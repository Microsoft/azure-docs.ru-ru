---
title: Настройка статического исходящего IP-адреса
description: Настройка брандмауэра Azure и определяемых пользователем маршрутов для рабочих нагрузок службы "экземпляры контейнеров Azure", использующих общедоступный IP-адрес брандмауэра для входящих и исходящих данных
ms.topic: article
ms.date: 07/16/2020
ms.openlocfilehash: 497645b9fe7f908cc9b8b4d7ed0ba5e201570160
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "89566574"
---
# <a name="configure-a-single-public-ip-address-for-outbound-and-inbound-traffic-to-a-container-group"></a>Настройка единого общедоступного IP-адреса для исходящего и входящего трафика в группу контейнеров

Настройка [группы контейнеров](container-instances-container-groups.md) с внешним IP-адресом позволяет внешним клиентам использовать IP-адрес для доступа к контейнеру в группе. Например, браузер может получить доступ к веб-приложению, выполняющемуся в контейнере. Однако в настоящее время для исходящего трафика группа контейнеров использует другой IP-адрес. Этот IP-адрес исходящего трафика не предоставляется программным способом, что делает мониторинг группы контейнеров и настройку правил брандмауэра клиента более сложной.

В этой статье описаны действия по настройке группы контейнеров в [виртуальной сети](container-instances-virtual-network-concepts.md) , интегрированной с [брандмауэром Azure](../firewall/overview.md). Настроив определяемый пользователем маршрут к группе контейнеров и правилам брандмауэра, можно маршрутизировать и определять трафик к группе контейнеров и из нее. Входной и выходной группы контейнеров используют общедоступный IP-адрес брандмауэра. Один IP-адрес исходящего трафика может использоваться несколькими группами контейнеров, развернутыми в подсети виртуальной сети, делегированной в службу "экземпляры контейнеров Azure".

В этой статье для создания ресурсов для этого сценария используется Azure CLI:

* Группы контейнеров, развернутые в делегированной подсети [в виртуальной сети](container-instances-vnet.md) 
* Брандмауэр Azure, развернутый в сети со статическим общедоступным IP-адресом
* Определяемый пользователем маршрут в подсети "группы контейнеров"
* Правило NAT для приема брандмауэра и правила приложения для исходящего трафика

Затем вы проверяете входящие и исходящие данные из примеров групп контейнеров через брандмауэр.

## <a name="deploy-aci-in-a-virtual-network"></a>Развертывание ACI в виртуальной сети

В типичном случае у вас уже может быть виртуальная сеть Azure для развертывания группы контейнеров. В демонстрационных целях следующие команды создают виртуальную сеть и подсеть при создании группы контейнеров. Подсеть делегирована службе "экземпляры контейнеров Azure". 

Группа контейнеров запускает небольшое веб-приложение из `aci-helloworld` образа. Как показано в других статьях документации, этот образ упаковывает небольшое веб-приложение, написанное на Node.js, которое обслуживает статическую HTML-страницу.

При необходимости сначала создайте группу ресурсов Azure с помощью команды [AZ Group Create][az-group-create] . Пример:

```azurecli
az group create --name myResourceGroup --location eastus
```

Чтобы упростить следующие примеры команд, используйте переменную среды для имени группы ресурсов:

```console
export RESOURCE_GROUP_NAME=myResourceGroup
```

Создайте группу контейнеров с помощью команды [AZ Container Create][az-container-create] :

```azurecli
az container create \
  --name appcontainer \
  --resource-group $RESOURCE_GROUP_NAME \
  --image mcr.microsoft.com/azuredocs/aci-helloworld \
  --vnet aci-vnet \
  --vnet-address-prefix 10.0.0.0/16 \
  --subnet aci-subnet \
  --subnet-address-prefix 10.0.0.0/24
```

> [!TIP]
> Измените значение `--subnet address-prefix` для пространства IP-адресов, которое требуется в подсети. Наименьшая поддерживаемая подсеть —/29, которая предоставляет восемь IP-адресов. Некоторые IP-адреса зарезервированы для использования в Azure.

Для использования на более позднем этапе получите частный IP-адрес группы контейнеров, выполнив команду [AZ Container демонстрация] [AZ-Container-шоу]:

```azurecli
ACI_PRIVATE_IP="$(az container show --name appcontainer \
  --resource-group $RESOURCE_GROUP_NAME \
  --query ipAddress.ip --output tsv)"
```

## <a name="deploy-azure-firewall-in-network"></a>Развертывание брандмауэра Azure в сети

В следующих разделах описано, как развернуть брандмауэр Azure в виртуальной сети с помощью Azure CLI. Дополнительные сведения см. [в разделе Учебник. Развертывание и настройка брандмауэра Azure с помощью портал Azure](../firewall/deploy-cli.md).

Сначала используйте команду [AZ Network vnet подсеть Create][az-network-vnet-subnet-create] , чтобы добавить подсеть с именем азурефиреваллсубнет для брандмауэра. Азурефиреваллсубнет — это *Обязательное* имя этой подсети.

```azurecli
az network vnet subnet create \
  --name AzureFirewallSubnet \
  --resource-group $RESOURCE_GROUP_NAME \
  --vnet-name aci-vnet   \
  --address-prefix 10.0.1.0/26
```

Используйте следующие [Azure CLI команды](../firewall/deploy-cli.md) для создания брандмауэра в подсети.

Добавьте расширение брандмауэра в Azure CLI, если оно еще не установлено, с помощью команды [AZ Extension Add][az-extension-add] :

```azurecli
az extension add --name azure-firewall
```

Создайте ресурсы брандмауэра:

```azurecli
az network firewall create \
  --name myFirewall \
  --resource-group $RESOURCE_GROUP_NAME \
  --location eastus

az network public-ip create \
  --name fw-pip \
  --resource-group $RESOURCE_GROUP_NAME \
  --location eastus \
  --allocation-method static \
  --sku standard
    
az network firewall ip-config create \
  --firewall-name myFirewall \
  --name FW-config \
  --public-ip-address fw-pip \
  --resource-group $RESOURCE_GROUP_NAME \
  --vnet-name aci-vnet
```

Обновите конфигурацию брандмауэра с помощью команды [AZ Network Firewall Update][az-network-firewall-update] :

```azurecli
az network firewall update \
  --name myFirewall \
  --resource-group $RESOURCE_GROUP_NAME
```

Получите частный IP-адрес брандмауэра с помощью команды [AZ Network Firewall IP-config list][az-network-firewall-ip-config-list] . Этот частный IP-адрес используется в следующей команде.


```azurecli
FW_PRIVATE_IP="$(az network firewall ip-config list \
  --resource-group $RESOURCE_GROUP_NAME \
  --firewall-name myFirewall \
  --query "[].privateIpAddress" --output tsv)"
```
Получите общедоступный IP-адрес брандмауэра с помощью команды [AZ Network public-IP демонстрация][az-network-public-ip-show] . Этот общедоступный IP-адрес используется в следующей команде.

```azurecli
FW_PUBLIC_IP="$(az network public-ip show \
  --name fw-pip \
  --resource-group $RESOURCE_GROUP_NAME \
  --query ipAddress --output tsv)"
```

## <a name="define-user-defined-route-on-aci-subnet"></a>Определение определяемого пользователем маршрута в подсети ACI

Определите определенный для использования маршрут в подсети ACI, чтобы перенаправить трафик в брандмауэр Azure. Дополнительные сведения см. в разделе [Маршрутизация сетевого трафика](../virtual-network/tutorial-create-route-table-cli.md). 

### <a name="create-route-table"></a>Создание таблицы маршрутов

Сначала выполните следующую команду [AZ Network Route-Table Create][az-network-route-table-create] , чтобы создать таблицу маршрутов. Создайте таблицу маршрутов в том же регионе, что и виртуальная сеть.

```azurecli
az network route-table create \
  --name Firewall-rt-table \
  --resource-group $RESOURCE_GROUP_NAME \
  --location eastus \
  --disable-bgp-route-propagation true
```

### <a name="create-route"></a>Создать маршрут

Чтобы создать маршрут в таблице маршрутов, выполните команду [AZ Network-Route-Table Route Create][az-network-route-table-route-create] . Чтобы направить трафик в брандмауэр, задайте тип следующего прыжка `VirtualAppliance` и передайте частный IP-адрес брандмауэра в качестве адреса следующего прыжка.

```azurecli
az network route-table route create \
  --resource-group $RESOURCE_GROUP_NAME \
  --name DG-Route \
  --route-table-name Firewall-rt-table \
  --address-prefix 0.0.0.0/0 \
  --next-hop-type VirtualAppliance \
  --next-hop-ip-address $FW_PRIVATE_IP
```

### <a name="associate-route-table-to-aci-subnet"></a>Связывание таблицы маршрутов с подсетью ACI

Выполните команду [AZ Network vnet подсети Update][az-network-vnet-subnet-update] , чтобы связать таблицу маршрутов с подсетью, делегированной в службу "экземпляры контейнеров Azure".

```azurecli
az network vnet subnet update \
  --name aci-subnet \
  --resource-group $RESOURCE_GROUP_NAME \
  --vnet-name aci-vnet \
  --address-prefixes 10.0.0.0/24 \
  --route-table Firewall-rt-table
```

## <a name="configure-rules-on-firewall"></a>Настройка правил в брандмауэре

По умолчанию брандмауэр Azure запрещает (блокирует) входящий и исходящий трафик. 

### <a name="configure-nat-rule-on-firewall-to-aci-subnet"></a>Настройка правила NAT на брандмауэре в подсети ACI

Создайте [правило NAT](../firewall/rule-processing.md) на брандмауэре, чтобы перевести и отфильтровать входящий Интернет-трафик в контейнер приложения, который был запущен ранее в сети. Дополнительные сведения см. [в разделе Фильтрация входящего интернет трафика с помощью брандмауэра Azure ДНаТ](../firewall/tutorial-firewall-dnat.md) .

Создайте правило NAT и коллекцию с помощью команды [AZ Network Firewall NAT-правило Create][az-network-firewall-nat-rule-create] :

```azurecli
az network firewall nat-rule create \
  --firewall-name myFirewall \
  --collection-name myNATCollection \
  --action dnat \
  --name myRule \
  --protocols TCP \
  --source-addresses '*' \
  --destination-addresses $FW_PUBLIC_IP \
  --destination-ports 80 \
  --resource-group $RESOURCE_GROUP_NAME \
  --translated-address $ACI_PRIVATE_IP \
  --translated-port 80 \
  --priority 200
```

При необходимости добавьте правила NAT, чтобы отфильтровать трафик по другим IP-адресам в подсети. Например, другие группы контейнеров в подсети могут предоставлять IP-адреса для входящего трафика, иначе при перезапуске группе контейнеров могут быть назначены другие внутренние IP-адреса.

### <a name="create-outbound-application-rule-on-the-firewall"></a>Создание правила исходящего приложения в брандмауэре

Чтобы создать правило исходящего трафика в брандмауэре, выполните следующую команду: [AZ Network Firewall приложение-Rule Create][az-network-firewall-application-rule-create] . Этот пример правила разрешает доступ из подсети, делегированной в службу "экземпляры контейнеров Azure", к полному доменному имени `checkip.dyndns.org` . Доступ по протоколу HTTP к сайту используется на следующем шаге для подтверждения IP-адреса исходящего трафика из экземпляров контейнеров Azure.

```azurecli
az network firewall application-rule create \
  --collection-name myAppCollection \
  --firewall-name myFirewall \
  --name Allow-CheckIP \
  --protocols Http=80 Https=443 \
  --resource-group $RESOURCE_GROUP_NAME \
  --target-fqdns checkip.dyndns.org \
  --source-addresses 10.0.0.0/24 \
  --priority 200 \
  --action Allow
```

## <a name="test-container-group-access-through-the-firewall"></a>Проверка доступа к группе контейнеров через брандмауэр

В следующих разделах проверяется, правильно ли настроена подсеть, делегированная в службу "экземпляры контейнеров Azure", за брандмауэром Azure. Предыдущие шаги направляют входящий трафик в подсеть и исходящий трафик из подсети через брандмауэр.

### <a name="test-ingress-to-a-container-group"></a>Проверка входящего трафика в группу контейнеров

Протестируйте входящий доступ к *appcontainer* , работающему в виртуальной сети, перейдя по общедоступному IP-адресу брандмауэра. Ранее вы сохранили общедоступный IP-адрес в переменной $FW _PUBLIC_IP:

```bash
echo $FW_PUBLIC_IP
```

Она выводит выходные данные следующего вида:

```console
52.142.18.133
```

Если правило NAT в брандмауэре настроено должным образом, при вводе общедоступного IP-адреса брандмауэра в браузере вы увидите следующее:

:::image type="content" source="media/container-instances-egress-ip-address/aci-ingress-ip-address.png" alt-text="Перейти к общедоступному IP-адресу брандмауэра":::

### <a name="test-egress-from-a-container-group"></a>Проверка исходящего трафика из группы контейнеров


Разверните следующий образец контейнера в виртуальной сети. При запуске он отправляет один HTTP-запрос в `http://checkip.dyndns.org` , который отображает IP-адрес отправителя (IP-адрес выхода). Если правило приложения на брандмауэре настроено правильно, то возвращается общедоступный IP-адрес брандмауэра.

```azurecli
az container create \
  --resource-group $RESOURCE_GROUP_NAME \
  --name testegress \
  --image mcr.microsoft.com/azuredocs/aci-tutorial-sidecar \
  --command-line "curl -s http://checkip.dyndns.org" \
  --restart-policy OnFailure \
  --vnet aci-vnet \
  --subnet aci-subnet
```

Просмотрите журналы контейнеров, чтобы убедиться, что IP-адрес совпадает с общедоступным IP-адресом брандмауэра.

```azurecli
az container logs \
  --resource-group $RESOURCE_GROUP_NAME \
  --name testegress 
```

Она выводит выходные данные следующего вида:

```console
<html><head><title>Current IP Check</title></head><body>Current IP Address: 52.142.18.133</body></html>
```

## <a name="next-steps"></a>Дальнейшие действия

В этой статье вы настроите группы контейнеров в виртуальной сети за брандмауэром Azure. Вы настроили определяемые пользователем маршруты и NAT и правила приложений в брандмауэре. С помощью этой конфигурации вы настраиваете один статический IP-адрес для входящих и исходящих данных из экземпляров контейнеров Azure.

Дополнительные сведения об управлении трафиком и защите ресурсов Azure см. в документации по [брандмауэру Azure](../firewall/index.yml) .



[az-group-create]: /cli/azure/group#az-group-create
[az-container-create]: /cli/azure/container#az-container-create
[az-network-vnet-subnet-create]: /cli/azure/network/vnet/subnet#az-network-vnet-subnet-create
[az-extension-add]: /cli/azure/extension#az-extension-add
[az-network-firewall-update]: /cli/azure/ext/azure-firewall/network/firewall#ext-azure-firewall-az-network-firewall-update
[az-network-public-ip-show]: /cli/azure/network/public-ip/#az-network-public-ip-show
[az-network-route-table-create]:/cli/azure/network/route-table/#az-network-route-table-create
[az-network-route-table-route-create]: /cli/azure/network/route-table/route#az-network-route-table-route-create
[az-network-firewall-ip-config-list]: /cli/azure/ext/azure-firewall/network/firewall/ip-config#ext-azure-firewall-az-network-firewall-ip-config-list
[az-network-vnet-subnet-update]: /cli/azure/network/vnet/subnet#az-network-vnet-subnet-update
[az-container-exec]: /cli/azure/container#az-container-exec
[az-vm-create]: /cli/azure/vm#az-vm-create
[az-vm-open-port]: /cli/azure/vm#az-vm-open-port
[az-vm-list-ip-addresses]: /cli/azure/vm#az-vm-list-ip-addresses
[az-network-firewall-application-rule-create]: /cli/azure/ext/azure-firewall/network/firewall/application-rule#ext-azure-firewall-az-network-firewall-application-rule-create
[az-network-firewall-nat-rule-create]: /cli/azure/ext/azure-firewall/network/firewall/nat-rule#ext-azure-firewall-az-network-firewall-nat-rule-create






