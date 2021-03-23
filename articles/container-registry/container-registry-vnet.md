---
title: Ограничение доступа с помощью конечной точки службы
description: Ограничьте доступ к реестру контейнеров Azure, используя конечную точку службы в виртуальной сети Azure. Доступ к конечной точке службы является компонентом уровня служб Premium.
ms.topic: article
ms.date: 05/04/2020
ms.openlocfilehash: c49595ee4ee79aef264a87dd48bccd03f3d4f5a5
ms.sourcegitcommit: 2c1b93301174fccea00798df08e08872f53f669c
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/22/2021
ms.locfileid: "104773901"
---
# <a name="restrict-access-to-a-container-registry-using-a-service-endpoint-in-an-azure-virtual-network"></a>Ограничение доступа к реестру контейнеров с использованием конечной точки службы в виртуальной сети Azure

[Виртуальная сеть Azure](../virtual-network/virtual-networks-overview.md) обеспечивает безопасное и конфиденциальное сетевое подключение для Azure и локальных ресурсов. [Конечная точка службы](../virtual-network/virtual-network-service-endpoints-overview.md) позволяет защитить общедоступный IP-адрес реестра контейнеров, разрешая доступ к нему только из виртуальной сети. Такая конечная точка направляет трафик к ресурсу по оптимальному маршруту в магистральной сети Azure. В каждом запросе также передаются удостоверения виртуальной сети и подсети.

В этой статье показано, как настроить в виртуальной сети конечную точку службы реестра контейнеров (предварительная версия). 

> [!IMPORTANT]
> Реестр контейнеров Azure теперь поддерживает [Приватный канал Azure](container-registry-private-link.md), что позволяет размещать в реестре частные конечные точки из виртуальной сети. Частные конечные точки доступны только из виртуальной сети через частные IP-адреса. Мы рекомендуем в большинстве сетевых сценариев использовать частные конечные точки вместо конечных точек службы.

Настройка конечной точки службы для реестра доступна для реестра контейнеров с уровнем **Премиум**. Сведения об уровнях служб реестра и ограничениях см. в статье [Уровни служб Реестра контейнеров Azure](container-registry-skus.md).

## <a name="preview-limitations"></a>Ограничения предварительной версии

* Дальнейшая доработка конечных точек службы для реестра контейнеров Azure пока не планируется. Мы рекомендуем использовать вместо них [частные конечные точки](container-registry-private-link.md).
* Вы не сможете настроить конечные точки службы для реестра на портале Azure.
* В качестве узла для доступа к реестру контейнеров через конечную точку службы можно использовать только кластер [Службы Azure Kubernetes](../aks/intro-kubernetes.md) или [виртуальную машину Azure](../virtual-machines/linux/overview.md). *Другие службы Azure, в том числе Экземпляры контейнеров Azure, не поддерживаются.*
* Конечные точки службы для реестра контейнеров Azure не поддерживаются в облаке Azure для государственных организаций США или в облаке Azure для Китая.

[!INCLUDE [container-registry-scanning-limitation](../../includes/container-registry-scanning-limitation.md)]

## <a name="prerequisites"></a>Предварительные требования

* Для выполнения действий с Azure CLI, описанных в этой статье, потребуется Azure CLI версии 2.0.58 или более поздней. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI 2.0][azure-cli].

* Если у вас еще нет реестра контейнеров Azure, создайте его (обязательно с уровнем "Премиум") и отправьте в него из Docker Hub пример образа, например `hello-world`. Создать реестр можно на [портале Azure][quickstart-portal] или с помощью [Azure CLI][quickstart-cli]. 

* Если вы хотите ограничить доступ к реестру, используя конечную точку службы из другой подписки Azure, зарегистрируйте в такой подписке поставщик ресурсов для реестра контейнеров Azure. Пример:

  ```azurecli
  az account set --subscription <Name or ID of subscription of virtual network>

  az provider register --namespace Microsoft.ContainerRegistry
  ``` 

[!INCLUDE [Set up Docker-enabled VM](../../includes/container-registry-docker-vm-setup.md)]

## <a name="configure-network-access-for-registry"></a>Настройка сетевого доступа к реестру

В этом разделе вы настроите для реестра контейнеров доступ из подсети, расположенной в виртуальной сети Azure. Действия предоставляются с помощью Azure CLI.

### <a name="add-a-service-endpoint-to-a-subnet"></a>Добавление конечной точки службы в подсеть

При создании виртуальной машины Azure по умолчанию создает в той же группе ресурсов виртуальную сеть. Имя для этой виртуальной сети создается автоматически на основе имени виртуальной машины. Например, если вы присвоите виртуальной машине имя *myDockerVM*, то созданная по умолчанию виртуальная сеть будет называться *myDockerVMVNET*, а подсеть в ней — *myDockerVMSubnet*. Проверьте это с помощью команды [AZ Network vnet List][az-network-vnet-list] :

```azurecli
az network vnet list \
  --resource-group myResourceGroup \
  --query "[].{Name: name, Subnet: subnets[0].name}"
```

Выходные данные:

```console
[
  {
    "Name": "myDockerVMVNET",
    "Subnet": "myDockerVMSubnet"
  }
]
```

Выполните команду [az network vnet subnet update][az-network-vnet-subnet-update], чтобы добавить конечную точку службы **Microsoft.ContainerRegistry** в существующую подсеть. Подставьте имена виртуальной сети и подсети в следующую команду:

```azurecli
az network vnet subnet update \
  --name myDockerVMSubnet \
  --vnet-name myDockerVMVNET \
  --resource-group myResourceGroup \
  --service-endpoints Microsoft.ContainerRegistry
```

Выполните команду [az network vnet subnet show][az-network-vnet-subnet-show], чтобы получить идентификатор ресурса для этой подсети. Он потребуется вам на следующем шаге, чтобы настроить правило сетевого доступа.

```azurecli
az network vnet subnet show \
  --name myDockerVMSubnet \
  --vnet-name myDockerVMVNET \
  --resource-group myResourceGroup \
  --query "id"
  --output tsv
``` 

Выходные данные:

```
/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/myResourceGroup/providers/Microsoft.Network/virtualNetworks/myDockerVMVNET/subnets/myDockerVMSubnet
```

### <a name="change-default-network-access-to-registry"></a>Изменение сетевого доступа по умолчанию для реестра

По умолчанию реестр контейнеров Azure разрешает подключения от узлов в любой сети. Чтобы разрешить доступ только из выбранной сети, установите режим запрета доступа по умолчанию. Подставьте нужное имя реестра в следующей команде [az acr update][az-acr-update]:

```azurecli
az acr update --name myContainerRegistry --default-action Deny
```

### <a name="add-network-rule-to-registry"></a>Добавление сетевого правила для реестра

Выполните команду [az acr network-rule add][az-acr-network-rule-add], чтобы добавить в реестр правило сетевого доступа, которое разрешает доступ из подсети, в которой расположена виртуальная машина. Подставьте имя реестра контейнеров и идентификатор ресурса подсети в следующую команду: 

 ```azurecli
az acr network-rule add \
  --name mycontainerregistry \
  --subnet <subnet-resource-id>
```

## <a name="verify-access-to-the-registry"></a>Проверка доступа к реестру

Подождите несколько минут, чтобы конфигурация обновилась, а затем убедитесь, что виртуальная машина может обращаться к реестру контейнеров. Создайте SSH-подключение к виртуальной машине и выполните команду [az acr login][az-acr-login], чтобы войти в реестр. 

```bash
az acr login --name mycontainerregistry
```

Попробуйте выполнить какие-либо операции с реестром, например выполните команду `docker pull`, чтобы извлечь пример образа. Замените значения образа и тега реальными значениями из вашего реестра, а также добавьте в качестве префикса имя сервера для входа в реестр (в нижнем регистре):

```bash
docker pull mycontainerregistry.azurecr.io/hello-world:v1
``` 

Средство Docker должно успешно передать образ на виртуальную машину.

Этот пример демонстрирует настройку доступа к частному реестру контейнеров с помощью правила сетевого доступа. Однако доступ к реестру будет невозможен с узла входа, на котором не настроено правило сетевого доступа. При попытке войти с другого узла с помощью команды `az acr login` или `docker login` вы получите примерно такой результат, как показано ниже:

```Console
Error response from daemon: login attempt to https://xxxxxxx.azurecr.io/v2/ failed with status: 403 Forbidden
```

## <a name="restore-default-registry-access"></a>Восстановление доступа к реестру по умолчанию

Чтобы восстановить стандартный режим доступа к реестру, удалите все настроенные сетевые правила. Затем установите режим разрешения доступа по умолчанию. 

### <a name="remove-network-rules"></a>Удаление сетевых правил

Чтобы просмотреть список сетевых правил, настроенных для реестра, выполните следующую команду [az acr network-rule list][az-acr-network-rule-list]:

```azurecli
az acr network-rule list --name mycontainerregistry 
```

Для каждого настроенного правила выполните команду [az acr network-rule remove][az-acr-network-rule-remove], чтобы удалить его. Пример:

```azurecli
# Remove a rule that allows access for a subnet. Substitute the subnet resource ID.

az acr network-rule remove \
  --name mycontainerregistry \
  --subnet /subscriptions/ \
  xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/myResourceGroup/providers/Microsoft.Network/virtualNetworks/myDockerVMVNET/subnets/myDockerVMSubnet
```

### <a name="allow-access"></a>Разрешить доступ

Подставьте нужное имя реестра в следующей команде [az acr update][az-acr-update]:
```azurecli
az acr update --name myContainerRegistry --default-action Allow
```

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы создали все ресурсы Azure в одной группе ресурсов и они вам больше не нужны, вы можете удалить все ресурсы сразу одной командой [az group delete](/cli/azure/group).

```azurecli
az group delete --name myResourceGroup
```

## <a name="next-steps"></a>Дальнейшие действия

* Чтобы ограничить доступ к реестру, используя частную конечную точку в виртуальной сети, см. статью [Настройка Приватного канала Azure для реестра контейнеров Azure](container-registry-private-link.md).
* Если вам нужно настроить правила для доступа к реестру, расположенному за клиентским брандмауэром, воспользуйтесь руководством [Настройка правил для доступа к реестру контейнеров Azure за брандмауэром](container-registry-firewall-access-rules.md).


<!-- IMAGES -->

[acr-subnet-service-endpoint]: ./media/container-registry-vnet/acr-subnet-service-endpoint.png


<!-- LINKS - External -->
[terms-of-use]: https://azure.microsoft.com/support/legal/preview-supplemental-terms/
[kubectl]: https://kubernetes.io/docs/user-guide/kubectl/
[docker-linux]: https://docs.docker.com/engine/installation/#supported-platforms
[docker-login]: https://docs.docker.com/engine/reference/commandline/login/
[docker-mac]: https://docs.docker.com/docker-for-mac/
[docker-push]: https://docs.docker.com/engine/reference/commandline/push/
[docker-tag]: https://docs.docker.com/engine/reference/commandline/tag/
[docker-windows]: https://docs.docker.com/docker-for-windows/

<!-- LINKS - Internal -->
[azure-cli]: /cli/azure/install-azure-cli
[az-acr-create]: /cli/azure/acr#az-acr-create
[az-acr-show]: /cli/azure/acr#az-acr-show
[az-acr-repository-show]: /cli/azure/acr/repository#az-acr-repository-show
[az-acr-repository-list]: /cli/azure/acr/repository#az-acr-repository-list
[az-acr-login]: /cli/azure/acr#az-acr-login
[az-acr-network-rule-add]: /cli/azure/acr/network-rule/#az-acr-network-rule-add
[az-acr-network-rule-remove]: /cli/azure/acr/network-rule/#az-acr-network-rule-remove
[az-acr-network-rule-list]: /cli/azure/acr/network-rule/#az-acr-network-rule-list
[az-acr-run]: /cli/azure/acr#az-acr-run
[az-acr-update]: /cli/azure/acr#az-acr-update
[az-ad-sp-create-for-rbac]: /cli/azure/ad/sp#az-ad-sp-create-for-rbac
[az-group-create]: /cli/azure/group
[az-role-assignment-create]: /cli/azure/role/assignment#az-role-assignment-create
[az-vm-create]: /cli/azure/vm#az-vm-create
[az-network-vnet-subnet-show]: /cli/azure/network/vnet/subnet/#az-network-vnet-subnet-show
[az-network-vnet-subnet-update]: /cli/azure/network/vnet/subnet/#az-network-vnet-subnet-update
[az-network-vnet-subnet-show]: /cli/azure/network/vnet/subnet/#az-network-vnet-subnet-show
[az-network-vnet-list]: /cli/azure/network/vnet/#az-network-vnet-list
[quickstart-portal]: container-registry-get-started-portal.md
[quickstart-cli]: container-registry-get-started-azure-cli.md
[azure-portal]: https://portal.azure.com
