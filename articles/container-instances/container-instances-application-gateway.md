---
title: Статический IP-адрес для группы контейнеров
description: Создание группы контейнеров в виртуальной сети и использование шлюза приложений Azure для предоставления статического внешнего IP-адреса контейнерному веб-приложению
ms.topic: article
ms.date: 03/16/2020
ms.openlocfilehash: 0131780fdb04a71837d5ae9bf5498bf2bd499f8a
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98035059"
---
# <a name="expose-a-static-ip-address-for-a-container-group"></a>Предоставление статического IP-адреса для группы контейнеров

В этой статье представлен один из способов предоставления статического, общедоступного IP-адреса для [группы контейнеров](container-instances-container-groups.md) с помощью [шлюза приложений](../application-gateway/overview.md)Azure. Выполните следующие действия, если требуется статическая точка входа для внешнего контейнерного приложения, работающего в службе "экземпляры контейнеров Azure". 

В этой статье для создания ресурсов для этого сценария используется Azure CLI:

* Виртуальная сеть Azure.
* Группа контейнеров, развернутая [в виртуальной сети, в](container-instances-vnet.md) которой размещено небольшое веб-приложение.
* Шлюз приложений с общедоступным внешним IP-адресом, прослушивателем для размещения веб-сайта на шлюзе и маршрут к серверной группе контейнеров.

Пока выполняется шлюз приложений и группа контейнеров предоставляет устойчивый частный IP-адрес в делегированной подсети сети, Группа контейнеров доступна по этому общедоступному IP-адресу.

> [!NOTE]
> Плата за использование Azure для шлюза приложений зависит от времени, в течение которого шлюз подготавливается и доступен, а также от объема обрабатываемых данных. См. [цены](https://azure.microsoft.com/pricing/details/application-gateway/).

## <a name="create-virtual-network"></a>Создание виртуальной сети

В типичном случае у вас уже может быть виртуальная сеть Azure. Если у вас ее нет, создайте ее, как показано в следующих примерах команд. Виртуальной сети требуются отдельные подсети для шлюза приложений и группы контейнеров.

Если требуется, создайте группу ресурсов Azure. Пример:

```azureci
az group create --name myResourceGroup --location eastus
```

Создайте виртуальную сеть с помощью команды [az network vnet create][az-network-vnet-create]. Эта команда создает подсеть *myAGSubnet* в сети.

```azurecli
az network vnet create \
  --name myVNet \
  --resource-group myResourceGroup \
  --location eastus \
  --address-prefix 10.0.0.0/16 \
  --subnet-name myAGSubnet \
  --subnet-prefix 10.0.1.0/24
```

Создайте подсеть для серверной группы контейнеров с помощью команды [AZ Network vnet подсети Create][az-network-vnet-subnet-create] . Здесь он называется *мяЦисубнет*.

```azurecli
az network vnet subnet create \
  --name myACISubnet \
  --resource-group myResourceGroup \
  --vnet-name myVNet   \
  --address-prefix 10.0.2.0/24
```

Создайте статический ресурс общедоступного IP-адреса с помощью команды [AZ Network public-IP Create][az-network-public-ip-create] . На следующем шаге этот адрес будет настроен в качестве внешнего интерфейса шлюза приложений.

```azurecli
az network public-ip create \
  --resource-group myResourceGroup \
  --name myAGPublicIPAddress \
  --allocation-method Static \
  --sku Standard
```

## <a name="create-container-group"></a>Создание группы контейнеров

Выполните следующую команду [AZ Container Create][az-container-create] , чтобы создать группу контейнеров в виртуальной сети, настроенной на предыдущем шаге. 

Группа развертывается в подсети *мяЦисубнет* и содержит единственный экземпляр с именем *appcontainer* , который извлекает `aci-helloworld` образ. Как показано в других статьях документации, этот образ упаковывает небольшое веб-приложение, написанное на Node.js, которое обслуживает статическую HTML-страницу. 

```azurecli
az container create \
  --name appcontainer \
  --resource-group myResourceGroup \
  --image mcr.microsoft.com/azuredocs/aci-helloworld \
  --vnet myVNet \
  --subnet myACISubnet
```

При успешном развертывании группе контейнеров назначается частный IP-адрес в виртуальной сети. Например, выполните следующую команду [AZ Container показывать][az-container-show] , чтобы получить IP-адрес группы:

```azurecli
az container show \
  --name appcontainer --resource-group myResourceGroup \
  --query ipAddress.ip --output tsv
```

Выходные данные должны быть следующего вида: `10.0.2.4`.

Для использования на более позднем этапе сохраните IP-адрес в переменной среды:

```azurecli
ACI_IP=$(az container show \
  --name appcontainer \
  --resource-group myResourceGroup \
  --query ipAddress.ip --output tsv)
```

> [!IMPORTANT]
> Если группа контейнеров остановлена, запущена или перезапущена, частный IP-адрес группы контейнеров может измениться. В этом случае необходимо будет обновить конфигурацию шлюза приложений.

## <a name="create-application-gateway"></a>Создание шлюза приложений

Создайте шлюз приложений в виртуальной сети, выполнив действия, описанные в [кратком руководстве по шлюзу приложений](../application-gateway/quick-create-cli.md). Следующая команда [AZ Network Application-Gateway Create][az-network-application-gateway-create] создает шлюз с общедоступным внешним IP-адресом и маршрутом к группе контейнеров серверной части. Дополнительные сведения о параметрах шлюза см. в [документации по шлюзу приложений](../application-gateway/index.yml) .

```azurecli
az network application-gateway create \
  --name myAppGateway \
  --location eastus \
  --resource-group myResourceGroup \
  --capacity 2 \
  --sku Standard_v2 \
  --http-settings-protocol http \
  --public-ip-address myAGPublicIPAddress \
  --vnet-name myVNet \
  --subnet myAGSubnet \
  --servers "$ACI_IP" 
```


Создание шлюза приложений в Azure может занять до 15 минут. 

## <a name="test-public-ip-address"></a>Проверить общедоступный IP-адрес
  
Теперь вы можете проверить доступ к веб-приложению, работающему в группе контейнеров за шлюзом приложений.

Выполните команду [AZ Network public-IP демонстрация][az-network-public-ip-show] , чтобы получить интерфейсный общедоступный IP-адрес шлюза:

```azurecli
az network public-ip show \
--resource-group myresourcegroup \
--name myAGPublicIPAddress \
--query [ipAddress] \
--output tsv
```

Выходные данные — это общедоступный IP-адрес, аналогичный: `52.142.18.133` .

Чтобы просмотреть запущенное веб-приложение после успешной настройки, перейдите к общедоступному IP-адресу шлюза в браузере. Успешный доступ имеет следующий вид:

![Снимок экрана браузера: приложение, выполняющееся в экземпляре контейнера Azure](./media/container-instances-application-gateway/aci-app-app-gateway.png)

## <a name="next-steps"></a>Дальнейшие действия

* См. [шаблон](https://github.com/Azure/azure-quickstart-templates/tree/master/201-aci-wordpress-vnet) быстрого запуска для создания группы контейнеров с экземпляром контейнера WordPress в качестве внутреннего сервера за шлюзом приложений.
* Шлюз приложений также можно настроить с помощью сертификата для завершения SSL-запросов. См. [Обзор](../application-gateway/ssl-overview.md) и [учебник](../application-gateway/create-ssl-portal.md).
* В зависимости от вашего сценария рассмотрите возможность использования других решений для балансировки нагрузки Azure с помощью экземпляров контейнеров Azure. Например, с помощью [диспетчера трафика Azure](../traffic-manager/traffic-manager-overview.md) можно распределять трафик между несколькими экземплярами контейнеров и в нескольких регионах. См. [статью в блоге](https://aaronmsft.com/posts/azure-container-instances/).

[az-network-vnet-create]:  /cli/azure/network/vnet#az-network-vnet-create
[az-network-vnet-subnet-create]: /cli/azure/network/vnet/subnet#az-network-vnet-subnet-create
[az-network-public-ip-create]: /cli/azure/network/public-ip#az-network-public-ip-create
[az-network-public-ip-show]: /cli/azure/network/public-ip#az-network-public-ip-show
[az-network-application-gateway-create]: /cli/azure/network/application-gateway#az-network-application-gateway-create
[az-container-create]: /cli/azure/container#az-container-create
[az-container-show]: /cli/azure/container#az-container-show
