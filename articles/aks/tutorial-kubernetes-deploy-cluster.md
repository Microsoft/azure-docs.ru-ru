---
title: Руководство по Kubernetes в Azure. Развертывание кластера
description: В этом руководстве по Службе Azure Kubernetes (AKS) вы создаете кластер AKS и используете kubectl для подключения к главному узлу Kubernetes.
services: container-service
ms.topic: tutorial
ms.date: 01/12/2021
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: d7e931a55ec0a9d46a8b92d4353bd2de8edd8818
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107777839"
---
# <a name="tutorial-deploy-an-azure-kubernetes-service-aks-cluster"></a>Руководство по Развертывание кластера службы Azure Kubernetes (AKS)

Kubernetes предоставляет распределенную платформу для контейнерных приложений. AKS позволяет быстро создать готовый производственный кластер Kubernetes. В этом руководстве (третья часть из восьми) кластер Kubernetes развертывается в AKS. Вы узнаете, как выполнять следующие задачи:

> [!div class="checklist"]
> * Развертывание кластера Kubernetes AKS, который может проходить проверку подлинности в Реестре контейнеров Azure
> * Установка интерфейса командной строки Kubernetes (kubectl).
> * Настройка kubectl для подключения к кластеру AKS.

В дальнейших руководствах приложение Azure для голосования развертывается в кластере, масштабируется и обновляется.

## <a name="before-you-begin"></a>Перед началом

В предыдущих руководствах образ контейнера был создан и передан в экземпляр реестра контейнеров Azure. Если вы не выполнили эти действия, вы можете начать с раздела руководства 1 [Создание образов контейнеров][aks-tutorial-prepare-app].

Для выполнения задач из этого руководства требуется Azure CLI 2.0.53 или более поздней версии. Чтобы узнать версию, выполните команду `az --version`. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI 2.0][azure-cli-install].

## <a name="create-a-kubernetes-cluster"></a>Создание кластера Kubernetes

Кластеры AKS могут использовать управление доступом Kubernetes на основе ролей (Kubernetes RBAC). Эти элементы управления позволяют определять доступ к ресурсам на основе ролей, назначенных пользователям. Разрешения объединяются, если пользователю назначено несколько ролей. Разрешения могут охватывать одно пространство имен или весь кластер. По умолчанию Azure CLI автоматически включает Kubernetes RBAC при создании кластера AKS.

Создайте кластер AKS с помощью команды [az aks create][]. В следующем примере в группе ресурсов *myResourceGroup* создается кластер с именем *myAKSCluster*. Эта группа ресурсов была создана в [предыдущем руководстве][aks-tutorial-prepare-acr] в регионе *eastus*. В следующем примере не указан регион, поэтому кластер AKS также создается в регионе *eastus*. Дополнительные сведения см. в статье [Квоты, ограничения размера виртуальных машин и доступность по регионам в Службе Azure Kubernetes (AKS)][quotas-skus-regions].

Чтобы разрешить кластеру AKS взаимодействовать с другими ресурсами Azure, автоматически создается удостоверение кластера, так как вы его не определили. Этому удостоверению кластера [предоставляется право на извлечение образов][container-registry-integration] из экземпляра Реестра контейнеров Azure (ACR), созданного при работе с предыдущим руководством. Для успешного выполнения команды необходима роль **Владелец** или **Администратор учетной записи Azure** в подписке Azure.

```azurecli
az aks create \
    --resource-group myResourceGroup \
    --name myAKSCluster \
    --node-count 2 \
    --generate-ssh-keys \
    --attach-acr <acrName>
```

Чтобы роль **Владелец** или **Администратор учетной записи Azure** не требовалась, вы можете также вручную настроить субъект-службу для извлечения образов из ACR. Дополнительные сведения см. в статьях [Аутентификация в реестре контейнеров Azure с помощью субъектов-служб](../container-registry/container-registry-auth-service-principal.md) и [Проверка подлинности в Kubernetes с использованием секрета для извлечения](../container-registry/container-registry-auth-kubernetes.md). В качестве альтернативы вместо субъекта-службы можно использовать [управляемое удостоверение](use-managed-identity.md).

Через несколько минут развертывание завершится, и отобразятся сведения о развертывании AKS в формате JSON.

> [!NOTE]
> Чтобы обеспечить надежную работу кластера, необходимо запустить не менее двух узлов.

## <a name="install-the-kubernetes-cli"></a>Установка интерфейса командной строки Kubernetes

Для подключения к кластеру Kubernetes с локального компьютера используйте средство [kubectl][kubectl] (клиент командной строки Kubernetes).

Если вы используете Azure Cloud Shell, `kubectl` уже установлен. Его также можно установить локально с помощью команды [az aks install-cli][]:

```azurecli
az aks install-cli
```

## <a name="connect-to-cluster-using-kubectl"></a>Подключение к кластеру с помощью kubectl

Чтобы настроить `kubectl` на подключение к кластеру Kubernetes, выполните команду [az aks get-credentials][]. В следующем примере возвращаются учетные данные для кластера AKS с именем *myAKSCluster* в группе ресурсов *myResourceGroup*:

```azurecli
az aks get-credentials --resource-group myResourceGroup --name myAKSCluster
```

Чтобы проверить подключение к кластеру, используйте команду [kubectl get nodes][kubectl-get] для получения списка узлов кластера.

```
$ kubectl get nodes

NAME                                STATUS   ROLES   AGE     VERSION
aks-nodepool1-37463671-vmss000000   Ready    agent   2m37s   v1.18.10
aks-nodepool1-37463671-vmss000001   Ready    agent   2m28s   v1.18.10
```

## <a name="next-steps"></a>Дальнейшие действия

В этом руководстве вы развернули кластер Kubernetes в AKS и настроили `kubectl` для подключения к нему. Вы ознакомились с выполнением следующих задач:

> [!div class="checklist"]
> * Развертывание кластера Kubernetes AKS, который может проходить проверку подлинности в Реестре контейнеров Azure
> * Установка интерфейса командной строки Kubernetes (kubectl).
> * Настройка kubectl для подключения к кластеру AKS.

Перейдите к следующему руководству, чтобы узнать, как развертывать приложения в кластер.

> [!div class="nextstepaction"]
> [Запуск приложений в Kubernetes][aks-tutorial-deploy-app]

<!-- LINKS - external -->
[kubectl]: https://kubernetes.io/docs/user-guide/kubectl/
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get

<!-- LINKS - internal -->
[aks-tutorial-deploy-app]: ./tutorial-kubernetes-deploy-application.md
[aks-tutorial-prepare-acr]: ./tutorial-kubernetes-prepare-acr.md
[aks-tutorial-prepare-app]: ./tutorial-kubernetes-prepare-app.md
[az ad sp create-for-rbac]: /cli/azure/ad/sp#az_ad_sp_create_for_rbac
[az acr show]: /cli/azure/acr#az_acr_show
[az role assignment create]: /cli/azure/role/assignment#az_role_assignment_create
[az aks create]: /cli/azure/aks#az_aks_create
[az aks install-cli]: /cli/azure/aks#az_aks_install_cli
[az aks get-credentials]: /cli/azure/aks#az_aks_get_credentials
[azure-cli-install]: /cli/azure/install-azure-cli
[container-registry-integration]: ./cluster-container-registry-integration.md
[quotas-skus-regions]: quotas-skus-regions.md
