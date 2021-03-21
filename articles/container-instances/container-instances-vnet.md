---
title: Развертывание группы контейнеров в виртуальной сети Azure
description: Узнайте, как развернуть группу контейнеров в новой или существующей виртуальной сети Azure с помощью интерфейса командной строки Azure.
ms.topic: article
ms.date: 07/02/2020
ms.custom: devx-track-js, devx-track-azurecli
ms.openlocfilehash: b791d3f37809c2eca53f5a3cd34f7c44dd11ce40
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98028885"
---
# <a name="deploy-container-instances-into-an-azure-virtual-network"></a>Развертывание экземпляров контейнеров в виртуальной сети Azure

[Виртуальная сеть Azure](../virtual-network/virtual-networks-overview.md) обеспечивает безопасное и конфиденциальное сетевое подключение для Azure и локальных ресурсов. Развертывание групп контейнеров в виртуальной сети Azure позволяет контейнерам безопасно обмениваться данными с другими ресурсами в виртуальной сети.

В этой статье показано, как использовать команду [AZ Container Create][az-container-create] в Azure CLI для развертывания групп контейнеров в новой виртуальной сети или в существующей виртуальной сети. 

Сетевые сценарии и ограничения см. в статье [сценарии и ресурсы виртуальной сети для службы "экземпляры контейнеров Azure](container-instances-virtual-network-concepts.md)".

> [!IMPORTANT]
> Развертывание группы контейнеров в виртуальной сети общедоступно для контейнеров Linux в большинстве регионов, где доступны службы "экземпляры контейнеров Azure". Дополнительные сведения см. в разделе [регионы и доступность ресурсов][container-regions]. 

Примеры в этой статье отформатированы для оболочки bash. Если вы предпочитаете использовать другую оболочку, например PowerShell или командную строку, измените соответствующим образом символы продолжения строки.


## <a name="deploy-to-new-virtual-network"></a>Развертывание в новой виртуальной сети

Чтобы обеспечить развертывание в новой виртуальной сети и автоматическое создание сетевых ресурсов в Azure, укажите следующее при выполнении команды [az container create][az-container-create]:

* Имя виртуальной сети
* Префикс адреса виртуальной сети в формате CIDR.
* Имя подсети
* Префикс адреса подсети в формате CIDR.

Префиксы адресов виртуальной сети и подсети определяют адресное пространство соответственно для виртуальной сети и подсети. Эти значения представлены в нотации бесклассовой междоменной маршрутизации (CIDR), например `10.0.0.0/16`. Дополнительные сведения о работе с подсетями см. в статье [Создание, изменение или удаление виртуальной сети](../virtual-network/virtual-network-manage-subnet.md).

Развернув первую группу контейнеров с помощью этого метода, можно выполнить развертывание в той же подсети, указав имена виртуальной сети и подсети, или же сетевой профиль, автоматически созданный в Azure. Так как Azure делегирует подсети службе "Экземпляры контейнеров Azure", в этой подсети можно развернуть *только* группы контейнеров.

### <a name="example"></a>Пример

Следующая команда [AZ Container Create][az-container-create] задает параметры для новой виртуальной сети и подсети. Укажите имя группы ресурсов, созданной в регионе, где [доступны](container-instances-region-availability.md)развертывания группы контейнеров в виртуальной сети. Эта команда развертывает общедоступный контейнер Microsoft [ACI-HelloWorld][aci-helloworld] , который работает с небольшим Node.js веб-сервером, обслуживающим статическую страницу. В следующем разделе вы развернете вторую группу контейнеров в той же подсети и протестируете связь между двумя экземплярами контейнеров.

```azurecli
az container create \
  --name appcontainer \
  --resource-group myResourceGroup \
  --image mcr.microsoft.com/azuredocs/aci-helloworld \
  --vnet aci-vnet \
  --vnet-address-prefix 10.0.0.0/16 \
  --subnet aci-subnet \
  --subnet-address-prefix 10.0.0.0/24
```

Развертывание в новой виртуальной сети с помощью этого метода может занять несколько минут, пока создаются сетевые ресурсы. После первоначального развертывания дополнительные развертывания групп контейнеров в одной подсети выполняются быстрее.

## <a name="deploy-to-existing-virtual-network"></a>Развертывание в существующей виртуальной сети

Чтобы развернуть группу контейнеров в существующей виртуальной сети, сделайте следующее:

1. Создайте подсеть в существующей виртуальной сети, используйте существующую подсеть, в которой уже развернута группа контейнеров, или используйте существующую подсеть, очищенную от *всех* остальных ресурсов.
1. Разверните группу контейнеров с помощью команды [az container create][az-container-create] и укажите один из следующих вариантов настроек:
   * имя виртуальной сети и имя подсети;
   * идентификатор ресурса виртуальной сети и идентификатор ресурса подсети, которые позволяют использовать виртуальную сеть из другой группы ресурсов;
   * идентификатор или имя сетевого профиля, которое можно получить с помощью команды [az network profile list][az-network-profile-list].

### <a name="example"></a>Пример

В следующем примере выполняется развертывание второй группы контейнеров в той же подсети, которая была создана ранее, и проверяет связь между двумя экземплярами контейнера.

Сначала получите IP-адрес первой развернутой группы контейнеров — *appcontainer*:

```azurecli
az container show --resource-group myResourceGroup \
  --name appcontainer \
  --query ipAddress.ip --output tsv
```

В выходных данных отображается IP-адрес группы контейнеров в частной подсети. Пример:

```console
10.0.0.4
```

Теперь в качестве значения параметра `CONTAINER_GROUP_IP` укажите IP-адрес, полученный командой `az container show`, и выполните следующую команду `az container create`. Этот второй контейнер, *commchecker*, запускает образ на базе Alpine Linux и выполняет `wget` в отношении IP-адреса частной подсети первой группы контейнеров.

```azurecli
CONTAINER_GROUP_IP=<container-group-IP-address>

az container create \
  --resource-group myResourceGroup \
  --name commchecker \
  --image alpine:3.5 \
  --command-line "wget $CONTAINER_GROUP_IP" \
  --restart-policy never \
  --vnet aci-vnet \
  --subnet aci-subnet
```

Когда завершится развертывание второго контейнера, извлеките его журналы, чтобы просмотреть выходные данные выполненной им команды `wget`:

```azurecli
az container logs --resource-group myResourceGroup --name commchecker
```

Если второй контейнер успешно обменивается данными с первым, вывод будет выглядеть примерно так:

```console
Connecting to 10.0.0.4 (10.0.0.4:80)
index.html           100% |*******************************|  1663   0:00:00 ETA
```

Выходные данные журнала должны показывать, что программе `wget` удалось подключиться и скачать файл индекса из первого контейнера, используя его частный IP-адрес в локальной подсети. Сетевой трафик между двумя группами контейнеров оставался в пределах виртуальной сети.

### <a name="example---yaml"></a>Пример — YAML

Вы также можете развернуть группу контейнеров в существующей виртуальной сети с помощью файла YAML, [шаблона диспетчер ресурсов](https://github.com/Azure/azure-quickstart-templates/tree/master/101-aci-vnet
)или другого программного метода, например, с помощью пакета SDK для Python. 

Например, при использовании файла YAML можно выполнить развертывание в виртуальной сети с подсетью, делегированной в службу "экземпляры контейнеров Azure". Задайте следующие свойства.

* `ipAddress`: Параметры частного IP-адреса для группы контейнеров.
  * `ports` — открываемые порты, если таковые имеются.
  * `protocol` — протокол (TCP или UDP) для открытого порта.
* `networkProfile`: Параметры сети для виртуальной сети и подсети.
  * `id` — полный идентификатор ресурса Resource Manager для `networkProfile`.

Чтобы получить идентификатор сетевого профиля, выполните команду [AZ Network Profile List][az-network-profile-list] , указав имя группы ресурсов, которая содержит виртуальную сеть и делегированную подсеть.

``` azurecli
az network profile list --resource-group myResourceGroup \
  --query [0].id --output tsv
```

Пример результатов выполнения:

```console
/subscriptions/<Subscription ID>/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkProfiles/aci-network-profile-aci-vnet-aci-subnet
```

Получив идентификатор сетевого профиля, скопируйте следующий код YAML в новый файл с именем *vnet-deploy-aci.yaml*. В разделе `networkProfile` замените значение `id` только что полученным идентификатором, затем сохраните файл. Этот код YAML создает группу контейнеров с именем *appcontaineryaml* в виртуальной сети.

```YAML
apiVersion: '2019-12-01'
location: westus
name: appcontaineryaml
properties:
  containers:
  - name: appcontaineryaml
    properties:
      image: mcr.microsoft.com/azuredocs/aci-helloworld
      ports:
      - port: 80
        protocol: TCP
      resources:
        requests:
          cpu: 1.0
          memoryInGB: 1.5
  ipAddress:
    type: Private
    ports:
    - protocol: tcp
      port: '80'
  networkProfile:
    id: /subscriptions/<Subscription ID>/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkProfiles/aci-network-profile-aci-vnet-subnet
  osType: Linux
  restartPolicy: Always
tags: null
type: Microsoft.ContainerInstance/containerGroups
```

Разверните группу контейнеров с помощью команды [AZ Container Create][az-container-create] , указав имя файла YAML для `--file` параметра:

```azurecli
az container create --resource-group myResourceGroup \
  --file vnet-deploy-aci.yaml
```

После завершения развертывания выполните команду [AZ Container Показать][az-container-show] , чтобы отобразить ее состояние. Образец вывода:

```console
Name              ResourceGroup    Status    Image                                       IP:ports     Network    CPU/Memory       OsType    Location
----------------  ---------------  --------  ------------------------------------------  -----------  ---------  ---------------  --------  ----------
appcontaineryaml  myResourceGroup  Running   mcr.microsoft.com/azuredocs/aci-helloworld  10.0.0.5:80  Private    1.0 core/1.5 gb  Linux     westus
```

## <a name="clean-up-resources"></a>Очистка ресурсов

### <a name="delete-container-instances"></a>Удаление экземпляров контейнеров

Когда завершите работу с созданными экземплярами контейнеров, удалите их с помощью следующих команд:

```azurecli
az container delete --resource-group myResourceGroup --name appcontainer -y
az container delete --resource-group myResourceGroup --name commchecker -y
az container delete --resource-group myResourceGroup --name appcontaineryaml -y
```

### <a name="delete-network-resources"></a>Удаление сетевых ресурсов

В настоящее время эта функция требует нескольких дополнительных команд для удаления созданных ранее сетевых ресурсов. Если вы использовали примеры команд из предыдущих разделов этой статьи для создания виртуальной сети и подсети, можно удалить эти сетевые ресурсы с помощью следующего скрипта. В сценарии предполагается, что группа ресурсов содержит одну виртуальную сеть с одним сетевым профилем.

Перед выполнением скрипта задайте в качестве переменной `RES_GROUP` имя группы ресурсов, содержащей виртуальную сеть и подсеть, которые следует удалить. Обновите имя виртуальной сети, если вы не использовали `aci-vnet` предложенное ранее имя. Скрипт отформатирован для оболочки Bash. Если вы предпочитаете использовать другую оболочку, например PowerShell или командную строку, измените методы доступа и присваивание значения переменной соответствующим образом.

> [!WARNING]
> Этот скрипт удаляет ресурсы! Он удаляет виртуальную сеть и все подсети, которые в ней содержатся. Убедитесь, что вам больше *не* требуются ресурсы в этой виртуальной сети, включая все содержащиеся в ней подсети, прежде чем запускать этот скрипт. После удаления **эти ресурсы восстановить невозможно**.

```azurecli
# Replace <my-resource-group> with the name of your resource group
# Assumes one virtual network in resource group
RES_GROUP=<my-resource-group>

# Get network profile ID
# Assumes one profile in virtual network
NETWORK_PROFILE_ID=$(az network profile list --resource-group $RES_GROUP --query [0].id --output tsv)

# Delete the network profile
az network profile delete --id $NETWORK_PROFILE_ID -y

# Delete virtual network
az network vnet delete --resource-group $RES_GROUP --name aci-vnet
```

## <a name="next-steps"></a>Дальнейшие действия

Чтобы развернуть новую виртуальную сеть, подсеть, сетевой профиль и группу контейнеров с помощью шаблона Resource Manager, см. страницу [создания группы контейнеров Azure с виртуальной сетью](https://github.com/Azure/azure-quickstart-templates/tree/master/101-aci-vnet
).

<!-- IMAGES -->
[aci-vnet-01]: ./media/container-instances-vnet/aci-vnet-01.png

<!-- LINKS - External -->
[aci-helloworld]: https://hub.docker.com/_/microsoft-azuredocs-aci-helloworld

<!-- LINKS - Internal -->
[az-container-create]: /cli/azure/container#az-container-create
[az-container-show]: /cli/azure/container#az-container-show
[az-network-vnet-create]: /cli/azure/network/vnet#az-network-vnet-create
[az-network-profile-list]: /cli/azure/network/profile#az-network-profile-list
[container-regions]: container-instances-region-availability.md
