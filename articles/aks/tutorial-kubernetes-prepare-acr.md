---
title: Руководство по работе с Kubernetes в Azure. Создание реестра контейнеров
description: С помощью этого руководства Azure Kubernetes Service (AKS) вы создадите экземпляр Реестра контейнеров Azure и отправите образ контейнера примера приложения.
services: container-service
ms.topic: tutorial
ms.date: 01/31/2021
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: fd53fab577797ad8bfdbf29b4a6d219e61ee3ef4
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107764267"
---
# <a name="tutorial-deploy-and-use-azure-container-registry"></a>Руководство. Развертывание Реестра контейнеров Azure и его использование

Реестр контейнеров Azure (ACR) — это частный реестр для образов контейнеров. Частный реестр контейнеров позволяет безопасно выполнять сборку и развертывать приложения, а также пользовательский код. В этом руководстве (здесь представлена вторая его часть из семи) вы развернете экземпляр ACR и отправите образ контейнера в экземпляр. Вы узнаете, как выполнять следующие задачи:

> [!div class="checklist"]
> * создание экземпляра Реестра контейнеров Azure;
> * добавление тегов к образу контейнера для реестра контейнеров Azure;
> * отправка образа в реестр контейнеров Azure.
> * просмотр образов в реестре.

В последующих руководствах этот экземпляр ACR интегрируется с кластером Kubernetes в AKS, а приложение развертывается из образа.

## <a name="before-you-begin"></a>Перед началом

В [предыдущей части руководства][aks-tutorial-prepare-app] мы создали образ контейнера для простого приложения Azure для голосования. Если вы еще не создали образ приложения Azure для голосования, выполните инструкции из статьи [Create container images to be used with Azure Container Service][aks-tutorial-prepare-app] (Создание образов контейнеров с помощью службы контейнеров Azure).

Для выполнения задач из этого руководства требуется Azure CLI 2.0.53 или более поздней версии. Чтобы узнать версию, выполните команду `az --version`. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI 2.0][azure-cli-install].

## <a name="create-an-azure-container-registry"></a>Создание реестра контейнеров Azure

Для создания Реестра контейнеров Azure сначала необходимо создать группу ресурсов. Группа ресурсов Azure является логическим контейнером, в котором происходит развертывание ресурсов Azure и управление ими.

Создайте группу ресурсов с помощью команды [az group create][az-group-create]. В следующем примере создается группа ресурсов с именем *myResourceGroup* в регионе *eastus*.

```azurecli
az group create --name myResourceGroup --location eastus
```

Создайте экземпляр Реестра контейнеров Azure с помощью команды [az acr create][az-acr-create] и введите собственное имя реестра. Имя реестра должно быть уникальным в пределах Azure и содержать от 5 до 50 буквенно-цифровых символов. В дальнейшем в этом руководстве `<acrName>` будет использоваться как заполнитель имени реестра контейнеров. Укажите уникальное имя для реестра. SKU *Базовый* — это оптимизированная по стоимости точка входа для целей разработки, обеспечивающая баланс ресурсов хранения и пропускной способности.

```azurecli
az acr create --resource-group myResourceGroup --name <acrName> --sku Basic
```

## <a name="log-in-to-the-container-registry"></a>Вход в реестр контейнеров

Чтобы использовать экземпляр ACR, сначала необходимо войти. Используйте команду [az acr login][az-acr-login] и укажите уникальное имя, заданное для реестра контейнеров на предыдущем шаге.

```azurecli
az acr login --name <acrName>
```

После выполнения эта команда возвращает сообщение *Login Succeeded* (Вход выполнен).

## <a name="tag-a-container-image"></a>Добавление тега к образу контейнера

Чтобы просмотреть список сохраненных образов, используйте команду [docker images][docker-images]:

```console
docker images
```
В выходных данных команды выше приведен список текущих локальных образов:

```output
REPOSITORY                                     TAG                 IMAGE ID            CREATED             SIZE
mcr.microsoft.com/azuredocs/azure-vote-front   v1                  84b41c268ad9        7 minutes ago       944MB
mcr.microsoft.com/oss/bitnami/redis            6.0.8               3a54a920bb6c        2 days ago          103MB
tiangolo/uwsgi-nginx-flask                     python3.6           a16ce562e863        6 weeks ago         944MB
```

Чтобы использовать образ контейнера *azure-vote-front* с ACR, образ нужно отметить тегом с адресом сервера входа в реестр. Данный тег используется для маршрутизации при отправке образов контейнеров в реестр образов.

Чтобы получить адрес сервера входа, используйте команду [az acr list][az-acr-list] и запросите *loginServer*, как показано ниже:

```azurecli
az acr list --resource-group myResourceGroup --query "[].{acrLoginServer:loginServer}" --output table
```

Теперь пометьте тегом локальный образ *azure-vote-front*, используя адрес *acrLoginServer* реестра контейнеров. Чтобы указать номер версии образа, добавьте *:v1* в конец имени образа:

```console
docker tag mcr.microsoft.com/azuredocs/azure-vote-front:v1 <acrLoginServer>/azure-vote-front:v1
```

Чтобы убедиться, что теги применены, запустите команду [docker images][docker-images] еще раз.

```console
docker images
```

Образ помечен адресом экземпляра ACR и номером версии.

```
REPOSITORY                                      TAG                 IMAGE ID            CREATED             SIZE
mcr.microsoft.com/azuredocs/azure-vote-front    v1                  84b41c268ad9        16 minutes ago      944MB
mycontainerregistry.azurecr.io/azure-vote-front v1                  84b41c268ad9        16 minutes ago      944MB
mcr.microsoft.com/oss/bitnami/redis             6.0.8               3a54a920bb6c        2 days ago          103MB
tiangolo/uwsgi-nginx-flask                      python3.6           a16ce562e863        6 weeks ago         944MB
```

## <a name="push-images-to-registry"></a>Отправка образов в реестр

Теперь, когда образ *azure-vote-front* создан и помечен тегом, вы можете отправить его в экземпляр ACR. Используйте команду [docker push][docker-push] и предоставьте собственный адрес *acrLoginServer* для имени образа так:

```console
docker push <acrLoginServer>/azure-vote-front:v1
```

Отправка образа в ACR может занять несколько минут.

## <a name="list-images-in-registry"></a>Перечисление образов в реестре

Чтобы получить список образов, отправленных в экземпляр ACR, выполните команду [az acr repository list][az-acr-repository-list]. Укажите собственное `<acrName>` следующим образом:

```azurecli
az acr repository list --name <acrName> --output table
```

В следующем примере выводятся образы *azure-vote-front* по доступности в реестре:

```output
Result
----------------
azure-vote-front
```

Чтобы увидеть теги для конкретного образа, используйте команду [az acr repository show-tags][az-acr-repository-show-tags], как показано ниже:

```azurecli
az acr repository show-tags --name <acrName> --repository azure-vote-front --output table
```

В следующем примере показан образ *v1*, отмеченный на предыдущем шаге:

```output
Result
--------
v1
```

Теперь у вас есть образ контейнера, сохраненный в частном экземпляре Реестра контейнеров Azure. В следующем руководстве мы развернем этот образ из ACR в кластер Kubernetes.

## <a name="next-steps"></a>Дальнейшие действия

В этом руководстве вы создали Реестр контейнеров Azure и отправили образ для использования в кластере AKS. Вы научились выполнять следующие задачи:

> [!div class="checklist"]
> * создание экземпляра Реестра контейнеров Azure;
> * добавление тегов к образу контейнера для реестра контейнеров Azure;
> * отправка образа в реестр контейнеров Azure.
> * просмотр образов в реестре.

Перейдите к следующему руководству, чтобы узнать, как развертывать кластер Kubernetes в Azure.

> [!div class="nextstepaction"]
> [Развертывание кластера Kubernetes][aks-tutorial-deploy-cluster]

<!-- LINKS - external -->
[docker-images]: https://docs.docker.com/engine/reference/commandline/images/
[docker-push]: https://docs.docker.com/engine/reference/commandline/push/

<!-- LINKS - internal -->
[az-acr-create]: /cli/azure/acr
[az-acr-list]: /cli/azure/acr
[az-acr-login]: /cli/azure/acr#az_acr_login
[az-acr-list]: /cli/azure/acr#az_acr_list
[az-acr-repository-list]: /cli/azure/acr/repository
[az-acr-repository-show-tags]: /cli/azure/acr/repository
[az-group-create]: /cli/azure/group#az_group_create
[azure-cli-install]: /cli/azure/install-azure-cli
[aks-tutorial-deploy-cluster]: ./tutorial-kubernetes-deploy-cluster.md
[aks-tutorial-prepare-app]: ./tutorial-kubernetes-prepare-app.md
