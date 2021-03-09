---
title: Интеграция реестра контейнеров Azure с помощью службы Kubernetes Azure
description: Узнайте, как интегрировать службу Kubernetes Azure (AKS) с реестром контейнеров Azure (запись контроля доступа).
services: container-service
manager: gwallace
ms.topic: article
ms.date: 01/08/2021
ms.openlocfilehash: 19ece696dabc81e643e8a904d506d22e40eaa099
ms.sourcegitcommit: 15d27661c1c03bf84d3974a675c7bd11a0e086e6
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/09/2021
ms.locfileid: "102499158"
---
# <a name="authenticate-with-azure-container-registry-from-azure-kubernetes-service"></a>Аутентификация с помощью реестра контейнеров Azure из Службы Azure Kubernetes

При использовании реестра контейнеров Azure (ACR) со Службой Azure Kubernetes (AKS) необходимо установить механизм аутентификации. Эта операция реализуется как часть интерфейса командной строки и портала, предоставляя необходимые разрешения для записи контроля доступа. В этой статье приведены примеры настройки проверки подлинности между этими двумя службами Azure. 

Вы можете настроить AKS для интеграции записей контроля доступа в нескольких простых командах с Azure CLI. Эта интеграция назначает роль Акрпулл управляемому удостоверению, связанному с кластером AKS.

> [!NOTE]
> В этой статье описывается автоматическая проверка подлинности между AKS и записью контроля доступа. Если необходимо извлечь образ из закрытого внешнего реестра, используйте [секрет для извлечения образа][Image Pull Secret].

## <a name="before-you-begin"></a>Подготовка к работе

Для этих примеров требуются:

* Роль **владельца** или **администратора учетной записи Azure** в **подписке Azure**
* Azure CLI версии 2.7.0 или более поздней

Чтобы избежать необходимости в роли администратора или **владельца** **учетной записи Azure** , можно настроить управляемое удостоверение вручную или использовать существующее управляемое удостоверение для проверки ПОдлинности записи контроля доступа из AKS. Дополнительные сведения см. в статье [использование управляемого удостоверения Azure для проверки подлинности в реестре контейнеров Azure](../container-registry/container-registry-authentication-managed-identity.md).

## <a name="create-a-new-aks-cluster-with-acr-integration"></a>Создание нового кластера AKS с интеграцией записей контроля доступа

Вы можете настроить интеграцию AKS и записей контроля доступа во время первоначального создания кластера AKS.  Чтобы разрешить кластеру AKS взаимодействовать с записью контроля доступа, используется **управляемое удостоверение** Azure Active Directory. Следующая команда CLI позволяет авторизовать существующую запись контроля доступа в подписке и настроить соответствующую роль **акрпулл** для управляемого удостоверения. Укажите допустимые значения для параметров ниже.

```azurecli
# set this to the name of your Azure Container Registry.  It must be globally unique
MYACR=myContainerRegistry

# Run the following line to create an Azure Container Registry if you do not already have one
az acr create -n $MYACR -g myContainerRegistryResourceGroup --sku basic

# Create an AKS cluster with ACR integration
az aks create -n myAKSCluster -g myResourceGroup --generate-ssh-keys --attach-acr $MYACR
```

Кроме того, можно указать имя записи контроля доступа с помощью идентификатора ресурса записей контроля доступа в следующем формате:

`/subscriptions/\<subscription-id\>/resourceGroups/\<resource-group-name\>/providers/Microsoft.ContainerRegistry/registries/\<name\>`

> [!NOTE]
> Если вы используете запись контроля доступа, которая находится в другой подписке из кластера AKS, используйте идентификатор ресурса записи контроля доступа при присоединении или отсоединении кластера AKS.

```azurecli
az aks create -n myAKSCluster -g myResourceGroup --generate-ssh-keys --attach-acr /subscriptions/<subscription-id>/resourceGroups/myContainerRegistryResourceGroup/providers/Microsoft.ContainerRegistry/registries/myContainerRegistry
```

Выполнение этого шага может занять несколько минут.

## <a name="configure-acr-integration-for-existing-aks-clusters"></a>Настройка интеграции записей контроля доступа для существующих кластеров AKS

Интегрируйте существующую запись контроля доступа с существующими кластерами AKS, указав допустимые значения для записей **контроля учетных** записей с именем или записи **контроля доступа (ИД ресурса** ), как показано ниже.

```azurecli
az aks update -n myAKSCluster -g myResourceGroup --attach-acr <acr-name>
```

или

```azurecli
az aks update -n myAKSCluster -g myResourceGroup --attach-acr <acr-resource-id>
```

Вы также можете удалить интеграцию между записью контроля доступа и кластером AKS, выполнив следующие

```azurecli
az aks update -n myAKSCluster -g myResourceGroup --detach-acr <acr-name>
```

или

```azurecli
az aks update -n myAKSCluster -g myResourceGroup --detach-acr <acr-resource-id>
```

## <a name="working-with-acr--aks"></a>Работа с записью контроля доступа & AKS

### <a name="import-an-image-into-your-acr"></a>Импорт изображения в запись контроля доступа

Импортируйте образ из DOCKER Hub в запись контроля доступа, выполнив следующую команду:


```azurecli
az acr import  -n <acr-name> --source docker.io/library/nginx:latest --image nginx:v1
```

### <a name="deploy-the-sample-image-from-acr-to-aks"></a>Развертывание примера образа из записи контроля доступа в AKS

Убедитесь, что у вас есть правильные учетные данные AKS

```azurecli
az aks get-credentials -g myResourceGroup -n myAKSCluster
```

Создайте файл с именем запись **контроля доступа nginx. YAML** , содержащий следующее. Замените имя ресурса реестра на **имя записи контроля доступа**. Пример: *миконтаинеррегистри*.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx0-deployment
  labels:
    app: nginx0-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx0
  template:
    metadata:
      labels:
        app: nginx0
    spec:
      containers:
      - name: nginx
        image: <acr-name>.azurecr.io/nginx:v1
        ports:
        - containerPort: 80
```

Затем запустите это развертывание в кластере AKS:

```console
kubectl apply -f acr-nginx.yaml
```

Вы можете отслеживать развертывание, выполнив:

```console
kubectl get pods
```

Необходимо иметь два работающих модуля.

```output
NAME                                 READY   STATUS    RESTARTS   AGE
nginx0-deployment-669dfc4d4b-x74kr   1/1     Running   0          20s
nginx0-deployment-669dfc4d4b-xdpd6   1/1     Running   0          20s
```

### <a name="troubleshooting"></a>Устранение неполадок
* Выполните команду [AZ AKS Check-запись контроля](/cli/azure/aks#az_aks_check_acr) доступа, чтобы убедиться, что реестр доступен из кластера AKS.
* Дополнительные сведения о [диагностике записей контроля](../container-registry/container-registry-diagnostics-audit-logs.md) доступа
* Дополнительные сведения о [работоспособности записей контроля](../container-registry/container-registry-check-health.md) доступа

<!-- LINKS - external -->
[AKS AKS CLI]: /cli/azure/aks#az-aks-create
[Image Pull secret]: https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/