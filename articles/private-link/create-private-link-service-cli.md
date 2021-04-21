---
title: Краткое руководство. Создание службы Приватного канала Azure с помощью Azure CLI
description: В этом кратком руководстве показано, как создать службу Приватного канала Azure с помощью Azure CLI.
services: private-link
author: asudbring
ms.service: private-link
ms.topic: quickstart
ms.date: 01/22/2021
ms.author: allensu
ms.openlocfilehash: c8e32a56148326104c3514b8a2fdb5d6bbd3f00a
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107778487"
---
# <a name="quickstart-create-a-private-link-service-using-azure-cli"></a>Краткое руководство. Создание службы Приватного канала с помощью Azure CLI

Приступите к созданию службы приватного канала, которая ссылается на вашу службу.  Предоставьте Приватному каналу доступ к службе или ресурсу, которые развернуты за Azure Load Balancer (цен. категория "Стандартный").  Пользователи службы имеют закрытый доступ из своей виртуальной сети.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)] 

- Для работы с этим кратким руководством требуется Azure CLI версии 2.0.28 или более поздней. Если вы используете Azure Cloud Shell, последняя версия уже установлена.

## <a name="create-a-resource-group"></a>Создание группы ресурсов

Группа ресурсов Azure является логическим контейнером, в котором происходит развертывание ресурсов Azure и управление ими.

Создайте группу ресурсов с помощью команды [az group create](/cli/azure/group#az_group_create):

* с именем **CreatePrivLinkService-rg**; 
* в расположении **eastus**.

```azurecli-interactive
  az group create \
    --name CreatePrivLinkService-rg \
    --location eastus2

```

## <a name="create-an-internal-load-balancer"></a>Создание внутреннего балансировщика нагрузки

В этом разделе показано, как создать виртуальную сеть и внутренний экземпляр Azure Load Balancer.

### <a name="virtual-network"></a>Виртуальная сеть

В этом разделе показано, как создать виртуальную сеть и подсеть для размещения подсистемы балансировки нагрузки, которая используется для доступа к службе приватного канала.

Создайте виртуальную сеть с помощью команды [az network vnet create](/cli/azure/network/vnet#az_network_vnet_create):

* с именем **myVNet**;
* с префиксом подсети **10.1.0.0/16**;
* с именем подсети **mySubnet**;
* с префиксом подсети **10.1.0.0/24**;
* в группе ресурсов **CreatePrivLinkService-rg**;
* в расположении **eastus2**;
* с отключенными сетевыми политиками для службы "Приватный канал" в подсети.

```azurecli-interactive
  az network vnet create \
    --resource-group CreatePrivLinkService-rg \
    --location eastus2 \
    --name myVNet \
    --address-prefixes 10.1.0.0/16 \
    --subnet-name mySubnet \
    --subnet-prefixes 10.1.0.0/24

```

Чтобы изменить настройки подсети для отключения сетевых политик службы "Приватный канал", используйте команду [az network vnet subnet update](/cli/azure/network/vnet/subnet#az_network_vnet_subnet_update).

```azurecli-interactive
az network vnet subnet update \
    --name mySubnet \
    --resource-group CreatePrivLinkService-rg \
    --vnet-name myVNet \
    --disable-private-link-service-network-policies true
```

### <a name="create-standard-load-balancer"></a>Создание подсистемы балансировки нагрузки (цен. категория "Стандартный")

В этом разделе описано, как создать и настроить следующие компоненты подсистемы балансировки нагрузки:

  * интерфейсный пул IP-адресов, который получает входящий трафик в подсистеме балансировки нагрузки;
  * внутренний пул IP-адресов, на который интерфейсный пул отправляет трафик с балансировкой нагрузки;
  * проверка работоспособности, определяющая работоспособность внутренних экземпляров виртуальной машины;
  * правило подсистемы балансировки нагрузки, определяющее порядок распределения трафика между виртуальными машинами.

### <a name="create-the-load-balancer-resource"></a>Создание ресурса подсистемы балансировки нагрузки

С помощью команды [az network lb create](/cli/azure/network/lb#az_network_lb_create) создайте общедоступную подсистему балансировки нагрузки:

* с именем **myLoadBalancer**,
* пулом переднего плана **myFrontEnd**
* и серверным пулом **myBackEndPool**,
* связанную с виртуальной сетью **myVNet**
* с внутренней подсетью **mySubnet**.

```azurecli-interactive
  az network lb create \
    --resource-group CreatePrivLinkService-rg \
    --name myLoadBalancer \
    --sku Standard \
    --vnet-name myVnet \
    --subnet mySubnet \
    --frontend-ip-name myFrontEnd \
    --backend-pool-name myBackEndPool
```

### <a name="create-the-health-probe"></a>Создание зонда работоспособности

При пробе работоспособности выполняется проверка всех экземпляров виртуальной машины, чтобы убедиться, что они могут отправлять сетевой трафик. 

Виртуальная машина с неудачной пробой удаляется из подсистемы балансировки нагрузки и снова добавляется в нее после устранения сбоя.

Создайте пробу работоспособности с помощью команды [az network lb probe create](/cli/azure/network/lb/probe#az_network_lb_probe_create), которая:

* отслеживает работоспособность виртуальных машин;
* имеет имя **myHealthProbe**;
* использует протокол **TCP**;
* отслеживает **порт 80**.

```azurecli-interactive
  az network lb probe create \
    --resource-group CreatePrivLinkService-rg \
    --lb-name myLoadBalancer \
    --name myHealthProbe \
    --protocol tcp \
    --port 80
```

### <a name="create-the-load-balancer-rule"></a>Создание правила подсистемы балансировки нагрузки

Правило подсистемы балансировки нагрузки определяет:

* конфигурацию интерфейсных IP-адресов для входящего трафика;
* серверный пул IP-адресов для приема трафика;
* требуемые порты источника и назначения. 

С помощью команды [az network lb rule create](/cli/azure/network/lb/rule#az_network_lb_rule_create) создайте правило подсистемы балансировки нагрузки, которое:

* имеет имя **myHTTPRule**;
* ожидает передачи данных от **порта 80**, используемого интерфейсным пулом **myFrontEnd**;
* перенаправляет трафик, для которого настроена балансировка нагрузки, ко внутреннему пулу адресов **myBackEndPool**, который использует **порт 80**; 
* использует пробу работоспособности **myHealthProbe**;
* использует протокол **TCP**;
* с временем ожидания перед переходом в режим простоя **15 минут**;
* и включенной функцией сброса подключений TCP.

```azurecli-interactive
  az network lb rule create \
    --resource-group CreatePrivLinkService-rg \
    --lb-name myLoadBalancer \
    --name myHTTPRule \
    --protocol tcp \
    --frontend-port 80 \
    --backend-port 80 \
    --frontend-ip-name myFrontEnd \
    --backend-pool-name myBackEndPool \
    --probe-name myHealthProbe \
    --idle-timeout 15 \
    --enable-tcp-reset true
```

## <a name="create-a-private-link-service"></a>Создание службы "Приватный канал"

В этом разделе описано, как создать службу "Приватный канал", которая использует созданный на предыдущем шаге Azure Load Balancer.

Создайте службу "Приватный канал", используя интерфейсную IP-конфигурацию Load Balancer (цен. категория "Стандартный") с помощью команды [az network private-link-service create](/cli/azure/network/private-link-service#az_network_private_link_service_create):

* с именем **myPrivateLinkService**;
* виртуальная сеть **myVNet**;
* связанную с Load Balancer (цен. категория "Стандартный") **myLoadBalancer** и интерфейсной конфигурацией **myFrontEnd**;
* в расположении **eastus2**.
 
```azurecli-interactive
az network private-link-service create \
    --resource-group CreatePrivLinkService-rg \
    --name myPrivateLinkService \
    --vnet-name myVNet \
    --subnet mySubnet \
    --lb-name myLoadBalancer \
    --lb-frontend-ip-configs myFrontEnd \
    --location eastus2
```

Ваша служба приватного канала создана и может принимать трафик. Если вы хотите просмотреть потоки трафика, настройте приложение за подсистемой Load Balancer ценовой категории "Стандартный".


## <a name="create-private-endpoint"></a>Создание частной конечной точки

В этом разделе вы сопоставите службу приватного канала с частной конечной точкой. Виртуальная сеть содержит частную конечную точку для службы приватного канала. Эта виртуальная сеть содержит ресурсы, которые будут обращаться к службе приватного канала.

### <a name="create-private-endpoint-virtual-network"></a>Создание виртуальной сети с частной конечной точкой

Создайте виртуальную сеть с помощью команды [az network vnet create](/cli/azure/network/vnet#az_network_vnet_create):

* с именем **myVNetPE**;
* с префиксом адреса **11.1.0.0/16**;
* с именем подсети **mySubnetPE**;
* с префиксом подсети **11.1.0.0/24**;
* в группе ресурсов **CreatePrivLinkService-rg**;
* в расположении **eastus2**.

```azurecli-interactive
  az network vnet create \
    --resource-group CreatePrivLinkService-rg \
    --location eastus2 \
    --name myVNetPE \
    --address-prefixes 11.1.0.0/16 \
    --subnet-name mySubnetPE \
    --subnet-prefixes 11.1.0.0/24
```

Чтобы изменить настройки подсети для отключения сетевых политик частной конечной точки, используйте команду [az network vnet subnet update](/cli/azure/network/vnet/subnet#az_network_vnet_subnet_update).

```azurecli-interactive
az network vnet subnet update \
    --name mySubnetPE \
    --resource-group CreatePrivLinkService-rg \
    --vnet-name myVNetPE \
    --disable-private-endpoint-network-policies true
```

### <a name="create-endpoint-and-connection"></a>Создание конечной точки и подключения

* Чтобы получить ИД ресурса для службы "Приватный канал", выполните команду [az network private-link-service show](/cli/azure/network/private-link-service#az_network_private_link_service_show). Команда помещает ИД ресурса в переменную для последующего использования.

* Создайте частную конечную точку в созданной ранее виртуальной сети с помощью команды [az network private-endpoint create](/cli/azure/network/private-endpoint#az_network_private_endpoint_create):

* с именем **MyPrivateEndpoint**;
* в группе ресурсов **CreatePrivLinkService-rg**;
* с именем подключения **myPEconnectiontoPLS**;
* в расположении **eastus2**;
* в виртуальной сети **myVNetPE** и подсети **mySubnetPE**.

```azurecli-interactive
  export resourceid=$(az network private-link-service show \
    --name myPrivateLinkService \
    --resource-group CreatePrivLinkService-rg \
    --query id \
    --output tsv)

  az network private-endpoint create \
    --connection-name myPEconnectiontoPLS \
    --name myPrivateEndpoint \
    --private-connection-resource-id $resourceid \
    --resource-group CreatePrivLinkService-rg \
    --subnet mySubnetPE \
    --manual-request false \
    --vnet-name myVNetPE 

```

## <a name="clean-up-resources"></a>Очистка ресурсов

Вы можете удалить ненужную группу ресурсов, службу "Приватный канал", подсистему балансировки нагрузки и все связанные с ней ресурсы, выполнив команду [az group delete](/cli/azure/group#az_group_delete).

```azurecli-interactive
  az group delete \
    --name CreatePrivLinkService-rg 
```

## <a name="next-steps"></a>Дальнейшие действия

В этом кратком руководстве показано, как:

* создать виртуальную сеть и внутренний экземпляр Azure Load Balancer;
* создать службу приватного канала.

Чтобы узнать больше о частной конечной точке Azure, ознакомьтесь со следующей статьей:
> [!div class="nextstepaction"]
> [Краткое руководство. Создание частной конечной точки с помощью Azure CLI](create-private-endpoint-cli.md)
