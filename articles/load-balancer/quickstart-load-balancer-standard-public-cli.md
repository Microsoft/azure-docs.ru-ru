---
title: Краткое руководство. Создание общедоступной подсистемы балансировки нагрузки, используя Azure СLI
titleSuffix: Azure Load Balancer
description: В рамках этого краткого руководства вы узнаете, как создать общедоступную подсистему балансировки нагрузки с помощью Azure CLI
services: load-balancer
documentationcenter: na
author: asudbring
manager: KumudD
tags: azure-resource-manager
ms.service: load-balancer
ms.devlang: na
ms.topic: quickstart
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 11/23/2020
ms.author: allensu
ms.custom: mvc, devx-track-js, devx-track-azurecli
ms.openlocfilehash: 9b332b18930d58ebb1d155c35a74eed69a90ce73
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107788787"
---
# <a name="quickstart-create-a-public-load-balancer-to-load-balance-vms-using-azure-cli"></a>Краткое руководство. Создание общедоступной подсистемы балансировки нагрузки с помощью Azure CLI для распределения нагрузки между виртуальными машинами

Начните работу с Azure Load Balancer, создав с помощью Azure CLI общедоступную подсистему балансировки нагрузки и три виртуальные машины.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

- Для работы с этим кратким руководством требуется Azure CLI версии 2.0.28 или более поздней. Если вы используете Azure Cloud Shell, последняя версия уже установлена.

## <a name="create-a-resource-group"></a>Создание группы ресурсов

Группа ресурсов Azure является логическим контейнером, в котором происходит развертывание ресурсов Azure и управление ими.

Создайте группу ресурсов с помощью команды [az group create](/cli/azure/group#az_group_create):

* с именем **CreatePubLBQS-rg**. 
* в расположении **eastus**.

```azurecli-interactive
  az group create \
    --name CreatePubLBQS-rg \
    --location eastus
```
---

# <a name="standard-sku"></a>[**SKU "Стандартный"**](#tab/option-1-create-load-balancer-standard)

>[!NOTE]
>Для производственных рабочих нагрузок рекомендуется использовать подсистему балансировки нагрузки ценовой категории "Стандартный". Дополнительные сведения о доступных ценовых категориях см. в статье **[Номера SKU для Azure Load Balancer](skus.md)** .

:::image type="content" source="./media/quickstart-load-balancer-standard-public-portal/resources-diagram.png" alt-text="Ресурсы подсистемы балансировки нагрузки ценовой категории &quot;Стандартный&quot;, созданные для работы с этим кратким руководством." border="false":::

## <a name="configure-virtual-network---standard"></a>Настройка виртуальной сети, категория "Стандартный"

Прежде чем развертывать виртуальные машины и тестировать подсистему балансировки нагрузки, создайте вспомогательные ресурсы виртуальной сети.

### <a name="create-a-virtual-network"></a>Создание виртуальной сети

Создайте виртуальную сеть с помощью команды [az network vnet create](/cli/azure/network/vnet#az_network_vnet_createt):

* с именем **myVNet**;
* с префиксом подсети **10.1.0.0/16**;
* с именем подсети **myBackendSubnet**;
* с префиксом подсети **10.1.0.0/24**;
* в группе ресурсов **CreatePubLBQS-rg**;
* расположении **eastus**.

```azurecli-interactive
  az network vnet create \
    --resource-group CreatePubLBQS-rg \
    --location eastus \
    --name myVNet \
    --address-prefixes 10.1.0.0/16 \
    --subnet-name myBackendSubnet \
    --subnet-prefixes 10.1.0.0/24
```
### <a name="create-a-public-ip-address"></a>Создание общедоступного IP-адреса

Чтобы создать общедоступный IP-адрес узла-бастиона, воспользуйтесь командой [az network public-ip create](/cli/azure/network/public-ip#az_network_public_ip_create):

* создайте стандартный избыточный между зонами общедоступный IP-адрес с именем **myPublicIP**;
* в группе ресурсов **CreatePubLBQS-rg**.

```azurecli-interactive
az network public-ip create \
    --resource-group CreatePubLBQS-rg \
    --name myBastionIP \
    --sku Standard
```
### <a name="create-a-bastion-subnet"></a>Создание подсети бастиона

Используйте команду [az network vnet subnet create](/cli/azure/network/vnet/subnet#az_network_vnet_subnet_create), чтобы создать подсеть бастиона:

* с именем **AzureBastionSubnet**;
* с префиксом подсети **10.1.1.0/24**.
* в виртуальной сети **myVNet**;
* в группе ресурсов **CreatePubLBQS-rg**;

```azurecli-interactive
az network vnet subnet create \
    --resource-group CreatePubLBQS-rg \
    --name AzureBastionSubnet \
    --vnet-name myVNet \
    --address-prefixes 10.1.1.0/24
```

### <a name="create-bastion-host"></a>Создание узла-бастиона

Используйте команду [az network bastion create](/cli/azure/network/bastion#az_network_bastion_create) для создания узла-бастиона:

* с именем **myBastionHost**;
* в группе ресурсов **CreatePubLBQS-rg**.
* связанного с общедоступным IP-адресом **myBastionIP**;
* связанного с виртуальной сетью **myVNet**;
* в расположении **eastus**.

```azurecli-interactive
az network bastion create \
    --resource-group CreatePubLBQS-rg \
    --name myBastionHost \
    --public-ip-address myBastionIP \
    --vnet-name myVNet \
    --location eastus
```

Развертывание узла-бастиона может занять несколько минут.

### <a name="create-a-network-security-group"></a>Создание группы безопасности сети

Для подсистемы балансировки нагрузки уровня "Стандартный" у серверных виртуальных машин должны быть сетевые интерфейсы, связанные с группой безопасности сети. 

Создайте группу безопасности сети с помощью команды [az network nsg create](/cli/azure/network/nsg#az_network_nsg_create):

* с именем **myNSG**;
* в группе ресурсов **CreatePubLBQS-rg**.

```azurecli-interactive
  az network nsg create \
    --resource-group CreatePubLBQS-rg \
    --name myNSG
```

### <a name="create-a-network-security-group-rule"></a>Создание правила группы безопасности сети

Создайте правило группы безопасности сети с помощью команды [az network nsg rule create](/cli/azure/network/nsg/rule#az_network_nsg_rule_create), используя такие сведения:

* имя **myNSGRuleHTTP**;
* группа безопасности сети, созданная на предыдущем шаге, **myNSG**;
* в группе ресурсов **CreatePubLBQS-rg**;
* с протоколом **(*)** ;
* направление **Входящий**;
* источник **(*)** ;
* назначение **(*)** ;
* порт назначения **порт 80**;
* доступ **Разрешить**;
* приоритет **200**.

```azurecli-interactive
  az network nsg rule create \
    --resource-group CreatePubLBQS-rg \
    --nsg-name myNSG \
    --name myNSGRuleHTTP \
    --protocol '*' \
    --direction inbound \
    --source-address-prefix '*' \
    --source-port-range '*' \
    --destination-address-prefix '*' \
    --destination-port-range 80 \
    --access allow \
    --priority 200
```

## <a name="create-backend-servers---standard"></a>Создание внутренних серверов, категория "Стандартный"

В этом разделе показано, как создать:

* три сетевых интерфейса для виртуальных машин;
* три виртуальные машины, которые будут использоваться в качестве внутренних серверов для подсистемы балансировки нагрузки.

### <a name="create-network-interfaces-for-the-virtual-machines"></a>Создание сетевых интерфейсов для виртуальных машин

Создайте три сетевых интерфейса с помощью команды [az network nic create](/cli/azure/network/nic#az_network_nic_create), используя следующие сведения:

* имена **myNicVM1**, **myNicVM2** и **myNicVM3**;
* в группе ресурсов **CreatePubLBQS-rg**;
* виртуальная сеть **myVNet**;
* подсеть **myBackendSubnet**;
* группа безопасности сети **myNSG**;

```azurecli-interactive
  array=(myNicVM1 myNicVM2 myNicVM3)
  for vmnic in "${array[@]}"
  do
    az network nic create \
        --resource-group CreatePubLBQS-rg \
        --name $vmnic \
        --vnet-name myVNet \
        --subnet myBackEndSubnet \
        --network-security-group myNSG
  done
```

### <a name="create-virtual-machines"></a>Создание виртуальных машин

Создайте виртуальные машины с помощью команды [az vm create](/cli/azure/vm#az_vm_create), используя следующие сведения:

### <a name="vm1"></a>VM1
* имя **myVM1**;
* в группе ресурсов **CreatePubLBQS-rg**;
* подключена к сетевому интерфейсу **myNicVM1**;
* образ виртуальной машины **win2019datacenter**;
* в **зоне 1**.

```azurecli-interactive
  az vm create \
    --resource-group CreatePubLBQS-rg \
    --name myVM1 \
    --nics myNicVM1 \
    --image win2019datacenter \
    --admin-username azureuser \
    --zone 1 \
    --no-wait
```
#### <a name="vm2"></a>VM2
* имя **myVM2**;
* в группе ресурсов **CreatePubLBQS-rg**;
* подключена к сетевому интерфейсу **myNicVM2**;
* образ виртуальной машины **win2019datacenter**;
* в **зоне 2**.

```azurecli-interactive
  az vm create \
    --resource-group CreatePubLBQS-rg \
    --name myVM2 \
    --nics myNicVM2 \
    --image win2019datacenter \
    --admin-username azureuser \
    --zone 2 \
    --no-wait
```

#### <a name="vm3"></a>VM3
* имя **myVM3**;
* в группе ресурсов **CreatePubLBQS-rg**;
* с подключением к сетевому интерфейсу **myNicVM3**;
* образ виртуальной машины **win2019datacenter**;
* в **зоне 3**.

```azurecli-interactive
   az vm create \
    --resource-group CreatePubLBQS-rg \
    --name myVM3 \
    --nics myNicVM3 \
    --image win2019datacenter \
    --admin-username azureuser \
    --zone 3 \
    --no-wait
```
На развертывание виртуальных машин может потребоваться несколько минут.

[!INCLUDE [ephemeral-ip-note.md](../../includes/ephemeral-ip-note.md)]

## <a name="create-a-public-ip-address---standard"></a>Создание общедоступного IP-адреса, категория "Стандартный"

Для доступа к веб-приложению через Интернет подсистеме балансировки нагрузки требуется общедоступный IP-адрес. 

Используйте [az network public-ip create](/cli/azure/network/public-ip#az_network_public_ip_create), чтобы:

* создать стандартный избыточный между зонами общедоступный IP-адрес с именем **myPublicIP**;
* в группе ресурсов **CreatePubLBQS-rg**.

```azurecli-interactive
  az network public-ip create \
    --resource-group CreatePubLBQS-rg \
    --name myPublicIP \
    --sku Standard
```

Создайте избыточный между зонами общедоступный IP-адрес в зоне 1, используя приведенные ниже сведения:

```azurecli-interactive
  az network public-ip create \
    --resource-group CreatePubLBQS-rg \
    --name myPublicIP \
    --sku Standard \
    --zone 1
```

## <a name="create-standard-load-balancer"></a>Создание подсистемы балансировки нагрузки (цен. категория "Стандартный")

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
* связанным с общедоступным IP-адресом **myPublicIP**, созданным на предыдущем шаге. 

```azurecli-interactive
  az network lb create \
    --resource-group CreatePubLBQS-rg \
    --name myLoadBalancer \
    --sku Standard \
    --public-ip-address myPublicIP \
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
    --resource-group CreatePubLBQS-rg \
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
    --resource-group CreatePubLBQS-rg \
    --lb-name myLoadBalancer \
    --name myHTTPRule \
    --protocol tcp \
    --frontend-port 80 \
    --backend-port 80 \
    --frontend-ip-name myFrontEnd \
    --backend-pool-name myBackEndPool \
    --probe-name myHealthProbe \
    --disable-outbound-snat true \
    --idle-timeout 15 \
    --enable-tcp-reset true

```
### <a name="add-virtual-machines-to-load-balancer-backend-pool"></a>Добавление виртуальных машин во внутренний пул подсистемы балансировки нагрузки

Добавьте виртуальные машины во внутренний пул, используя команду [az network nic ip-config address-pool add](/cli/azure/network/nic/ip-config/address-pool#az_network_nic_ip_config_address_pool_add):

* в серверном пуле адресов **myBackEndPool**;
* в группе ресурсов **CreatePubLBQS-rg**;
* связанный с подсистемой балансировки нагрузки **myLoadBalancer**.

```azurecli-interactive
  array=(myNicVM1 myNicVM2 myNicVM3)
  for vmnic in "${array[@]}"
  do
    az network nic ip-config address-pool add \
     --address-pool myBackendPool \
     --ip-config-name ipconfig1 \
     --nic-name $vmnic \
     --resource-group CreatePubLBQS-rg \
     --lb-name myLoadBalancer
  done
```

## <a name="create-outbound-rule-configuration"></a>Создание конфигурации правила для исходящего трафика
Правила для исходящего трафика подсистемы балансировки нагрузки настраивают исходящий SNAT для виртуальных машин во внутреннем пуле. 

Дополнительные сведения об исходящих подключениях см. в статье [Исходящие подключения в Azure](load-balancer-outbound-connections.md).

При настройке исходящего трафика можно использовать общедоступный IP-адрес или префикс.

### <a name="public-ip"></a>Общедоступный IP-адрес

Используйте команду [az network public-ip create](/cli/azure/network/public-ip#az_network_public_ip_create), чтобы создать отдельный IP-адрес для исходящего подключения.  

* с именем **myPublicIPOutbound**;
* в группе ресурсов **CreatePubLBQS-rg**.

```azurecli-interactive
  az network public-ip create \
    --resource-group CreatePubLBQS-rg \
    --name myPublicIPOutbound \
    --sku Standard
```

Создайте избыточный между зонами общедоступный IP-адрес в зоне 1, используя приведенные ниже сведения:

```azurecli-interactive
  az network public-ip create \
    --resource-group CreatePubLBQS-rg \
    --name myPublicIPOutbound \
    --sku Standard \
    --zone 1
```

### <a name="public-ip-prefix"></a>Префикс общедоступного IP-адреса

Используйте команду [az network public-ip prefix create](/cli/azure/network/public-ip/prefix#az_network_public_ip_prefix_create), чтобы создать префикс общедоступного IP-адреса для исходящего подключения.

* с именем **myPublicIPPrefixOutbound**;
* в группе ресурсов **CreatePubLBQS-rg**.
* длина префикса **28**.

```azurecli-interactive
  az network public-ip prefix create \
    --resource-group CreatePubLBQS-rg \
    --name myPublicIPPrefixOutbound \
    --length 28
```
Создайте зональный общедоступный IP-адрес в зоне доступности 1, используя приведенные ниже сведения:

```azurecli-interactive
  az network public-ip prefix create \
    --resource-group CreatePubLBQS-rg \
    --name myPublicIPPrefixOutbound \
    --length 28 \
    --zone 1
```

Дополнительные сведения о масштабировании исходящего преобразования сетевых адресов (NAT) и исходящего подключения см. в разделе [Масштабирование NAT для исходящего трафика при использовании нескольких IP-адресов](load-balancer-outbound-connections.md).

### <a name="create-outbound-frontend-ip-configuration"></a>Создайте интерфейсные IP-конфигурации для исходящего трафика

Создайте новую интерфейсную IP-конфигурацию с помощью [az network lb frontend-ip create ](/cli/azure/network/lb/frontend-ip#az_network_lb_frontend_ip_create):

Выберите команды "общедоступный IP-адрес" или "префикс общедоступного IP-адреса" на основе решения на предыдущем шаге.

#### <a name="public-ip"></a>Общедоступный IP-адрес

* с именем **myFrontEndOutbound**;
* в группе ресурсов **CreatePubLBQS-rg**;
* связанный с общедоступным IP-адресом **myPublicIPOutbound**;
* связанный с подсистемой балансировки нагрузки **myLoadBalancer**.

```azurecli-interactive
  az network lb frontend-ip create \
    --resource-group CreatePubLBQS-rg \
    --name myFrontEndOutbound \
    --lb-name myLoadBalancer \
    --public-ip-address myPublicIPOutbound 
```

#### <a name="public-ip-prefix"></a>Префикс общедоступного IP-адреса

* с именем **myFrontEndOutbound**;
* в группе ресурсов **CreatePubLBQS-rg**;
* связанный с префиксом общедоступного IP-адреса **myPublicIPPrefixOutbound**;
* связанный с подсистемой балансировки нагрузки **myLoadBalancer**.

```azurecli-interactive
  az network lb frontend-ip create \
    --resource-group CreatePubLBQS-rg \
    --name myFrontEndOutbound \
    --lb-name myLoadBalancer \
    --public-ip-prefix myPublicIPPrefixOutbound 
```

### <a name="create-outbound-pool"></a>Создание исходящего пула

Создайте исходящий пул, используя команду [az network lb address-pool create](/cli/azure/network/lb/address-pool#az_network_lb_address_pool_create) и cледующие сведения:

* имя **myBackEndPoolOutbound**;
* в группе ресурсов **CreatePubLBQS-rg**;
* связанный с подсистемой балансировки нагрузки **myLoadBalancer**.

```azurecli-interactive
  az network lb address-pool create \
    --resource-group CreatePubLBQS-rg \
    --lb-name myLoadBalancer \
    --name myBackendPoolOutbound
```
### <a name="create-outbound-rule"></a>Создание правила для исходящего трафика

Создайте правило для исходящего трафика для исходящего внутреннего пула, используя команду [az network lb outbound-rule create](/cli/azure/network/lb/outbound-rule#az_network_lb_outbound_rule_create) и следующие сведения:

* имя **myOutboundRule**;
* в группе ресурсов **CreatePubLBQS-rg**;
* связь с подсистемой балансировки нагрузки **myLoadBalancer**;
* связь с внешним интерфейсом **myFrontEndOutbound**;
* протокол **Все**;
* время ожидания перед переходом в режим простоя **15**;
* исходящие порты **10000**;
* связь с серверным пулом **myBackEndPoolOutbound**.

```azurecli-interactive
  az network lb outbound-rule create \
    --resource-group CreatePubLBQS-rg \
    --lb-name myLoadBalancer \
    --name myOutboundRule \
    --frontend-ip-configs myFrontEndOutbound \
    --protocol All \
    --idle-timeout 15 \
    --outbound-ports 10000 \
    --address-pool myBackEndPoolOutbound
```
### <a name="add-virtual-machines-to-outbound-pool"></a>Добавление виртуальных машин в исходящий пул

Добавьте виртуальные машины в исходящий пул, используя команду [az network nic ip-config address-pool add](/cli/azure/network/nic/ip-config/address-pool#az_network_nic_ip_config_address_pool_add) и следующие сведения:


* в серверном пуле адресов **myBackEndPoolOutbound**;
* в группе ресурсов **CreatePubLBQS-rg**;
* связанный с подсистемой балансировки нагрузки **myLoadBalancer**.

```azurecli-interactive
  array=(myNicVM1 myNicVM2 myNicVM3)
  for vmnic in "${array[@]}"
  do
    az network nic ip-config address-pool add \
     --address-pool myBackendPoolOutbound \
     --ip-config-name ipconfig1 \
     --nic-name $vmnic \
     --resource-group CreatePubLBQS-rg \
     --lb-name myLoadBalancer
  done
```

# <a name="basic-sku"></a>[**SKU "Базовый"**](#tab/option-1-create-load-balancer-basic)

>[!NOTE]
>Для производственных рабочих нагрузок рекомендуется использовать подсистему балансировки нагрузки ценовой категории "Стандартный". Дополнительные сведения о доступных ценовых категориях см. в статье **[Номера SKU для Azure Load Balancer](skus.md)** .

:::image type="content" source="./media/quickstart-load-balancer-standard-public-portal/resources-diagram-basic.png" alt-text="Ресурсы подсистемы балансировки нагрузки ценовой категории &quot;Базовый&quot;, созданные при работе с этим кратким руководством." border="false":::

## <a name="configure-virtual-network---basic"></a>Настройка виртуальной сети, категория "Базовый"

Прежде чем развертывать виртуальные машины и тестировать подсистему балансировки нагрузки, создайте вспомогательные ресурсы виртуальной сети.

### <a name="create-a-virtual-network"></a>Создание виртуальной сети

Создайте виртуальную сеть с помощью команды [az network vnet create](/cli/azure/network/vnet#az_network_vnet_create):

* с именем **myVNet**;
* с префиксом подсети **10.1.0.0/16**;
* с именем подсети **myBackendSubnet**;
* с префиксом подсети **10.1.0.0/24**;
* в группе ресурсов **CreatePubLBQS-rg**;
* расположении **eastus**.

```azurecli-interactive
  az network vnet create \
    --resource-group CreatePubLBQS-rg \
    --location eastus \
    --name myVNet \
    --address-prefixes 10.1.0.0/16 \
    --subnet-name myBackendSubnet \
    --subnet-prefixes 10.1.0.0/24
```

### <a name="create-a-public-ip-address"></a>Создание общедоступного IP-адреса

Чтобы создать общедоступный IP-адрес узла-бастиона, воспользуйтесь командой [az network public-ip create](/cli/azure/network/public-ip#az_network_public_ip_create):

* создайте стандартный избыточный между зонами общедоступный IP-адрес с именем **myPublicIP**;
* в группе ресурсов **CreatePubLBQS-rg**.

```azurecli-interactive
az network public-ip create \
    --resource-group CreatePubLBQS-rg \
    --name myBastionIP \
    --sku Standard
```
### <a name="create-a-bastion-subnet"></a>Создание подсети бастиона

Используйте команду [az network vnet subnet create](/cli/azure/network/vnet/subnet#az_network_vnet_subnet_create), чтобы создать подсеть бастиона:

* с именем **AzureBastionSubnet**;
* с префиксом подсети **10.1.1.0/24**.
* в виртуальной сети **myVNet**;
* в группе ресурсов **CreatePubLBQS-rg**;

```azurecli-interactive
az network vnet subnet create \
    --resource-group CreatePubLBQS-rg \
    --name AzureBastionSubnet \
    --vnet-name myVNet \
    --address-prefixes 10.1.1.0/24
```

### <a name="create-bastion-host"></a>Создание узла-бастиона

Используйте команду [az network bastion create](/cli/azure/network/bastion#az_network_bastion_create) для создания узла-бастиона:

* с именем **myBastionHost**;
* в группе ресурсов **CreatePubLBQS-rg**.
* связанного с общедоступным IP-адресом **myBastionIP**;
* связанного с виртуальной сетью **myVNet**;
* в расположении **eastus**.

```azurecli-interactive
az network bastion create \
    --resource-group CreatePubLBQS-rg \
    --name myBastionHost \
    --public-ip-address myBastionIP \
    --vnet-name myVNet \
    --location eastus
```

Развертывание узла-бастиона может занять несколько минут.

### <a name="create-a-network-security-group"></a>Создание группы безопасности сети

Для подсистемы балансировки нагрузки уровня "Стандартный" у серверных виртуальных машин должны быть сетевые интерфейсы, связанные с группой безопасности сети. 

Создайте группу безопасности сети с помощью команды [az network nsg create](/cli/azure/network/nsg#az_network_nsg_create):

* с именем **myNSG**;
* в группе ресурсов **CreatePubLBQS-rg**.

```azurecli-interactive
  az network nsg create \
    --resource-group CreatePubLBQS-rg \
    --name myNSG
```

### <a name="create-a-network-security-group-rule"></a>Создание правила группы безопасности сети

Создайте правило группы безопасности сети с помощью команды [az network nsg rule create](/cli/azure/network/nsg/rule#az_network_nsg_rule_create), используя такие сведения:

* имя **myNSGRuleHTTP**;
* группа безопасности сети, созданная на предыдущем шаге, **myNSG**;
* в группе ресурсов **CreatePubLBQS-rg**;
* с протоколом **(*)** ;
* направление **Входящий**;
* источник **(*)** ;
* назначение **(*)** ;
* порт назначения **порт 80**;
* доступ **Разрешить**;
* приоритет **200**.

```azurecli-interactive
  az network nsg rule create \
    --resource-group CreatePubLBQS-rg \
    --nsg-name myNSG \
    --name myNSGRuleHTTP \
    --protocol '*' \
    --direction inbound \
    --source-address-prefix '*' \
    --source-port-range '*' \
    --destination-address-prefix '*' \
    --destination-port-range 80 \
    --access allow \
    --priority 200
```

## <a name="create-backend-servers---basic"></a>Создание внутренних серверов, категория "Базовый"

В этом разделе показано, как создать:

* три сетевых интерфейса для виртуальных машин;
* группу доступности для виртуальных машин;
* три виртуальные машины, которые будут использоваться в качестве внутренних серверов для подсистемы балансировки нагрузки.


### <a name="create-network-interfaces-for-the-virtual-machines"></a>Создание сетевых интерфейсов для виртуальных машин

Создайте три сетевых интерфейса с помощью команды [az network nic create](/cli/azure/network/nic#az_network_nic_create), используя следующие сведения:


* имена **myNicVM1**, **myNicVM2** и **myNicVM3**;
* в группе ресурсов **CreatePubLBQS-rg**;
* виртуальная сеть **myVNet**;
* подсеть **myBackendSubnet**;
* группа безопасности сети **myNSG**;

```azurecli-interactive
  array=(myNicVM1 myNicVM2 myNicVM3)
  for vmnic in "${array[@]}"
  do
    az network nic create \
        --resource-group CreatePubLBQS-rg \
        --name $vmnic \
        --vnet-name myVNet \
        --subnet myBackEndSubnet \
        --network-security-group myNSG
  done
```
### <a name="create-availability-set-for-virtual-machines"></a>Создание группы доступности для виртуальных машин

Создайте группу доступности, используя команду [az vm availability-set create](/cli/azure/vm/availability-set#az_vm_availability_set_create) и следующие сведения:

* имя **myAvSet**;
* в группе ресурсов **CreatePubLBQS-rg**;
* расположение **eastus**.

```azurecli-interactive
  az vm availability-set create \
    --name myAvSet \
    --resource-group CreatePubLBQS-rg \
    --location eastus 
    
```

### <a name="create-virtual-machines"></a>Создание виртуальных машин

Создайте виртуальные машины с помощью команды [az vm create](/cli/azure/vm#az_vm_create), используя следующие сведения:

### <a name="vm1"></a>VM1
* имя **myVM1**;
* в группе ресурсов **CreatePubLBQS-rg**;
* подключена к сетевому интерфейсу **myNicVM1**;
* образ виртуальной машины **win2019datacenter**;
* в **зоне 1**.

```azurecli-interactive
  az vm create \
    --resource-group CreatePubLBQS-rg \
    --name myVM1 \
    --nics myNicVM1 \
    --image win2019datacenter \
    --admin-username azureuser \
    --availability-set myAvSet \
    --no-wait
```
#### <a name="vm2"></a>VM2
* имя **myVM2**;
* в группе ресурсов **CreatePubLBQS-rg**;
* подключена к сетевому интерфейсу **myNicVM2**;
* образ виртуальной машины **win2019datacenter**;
* в **зоне 2**.

```azurecli-interactive
  az vm create \
    --resource-group CreatePubLBQS-rg \
    --name myVM2 \
    --nics myNicVM2 \
    --image win2019datacenter \
    --admin-username azureuser \
    --availability-set myAvSet \
    --no-wait
```

#### <a name="vm3"></a>VM3
* имя **myVM3**;
* в группе ресурсов **CreatePubLBQS-rg**;
* с подключением к сетевому интерфейсу **myNicVM3**;
* образ виртуальной машины **win2019datacenter**;
* в **зоне 3**.

```azurecli-interactive
   az vm create \
    --resource-group CreatePubLBQS-rg \
    --name myVM3 \
    --nics myNicVM3 \
    --image win2019datacenter \
    --admin-username azureuser \
    --availability-set myAvSet \
    --no-wait
```
На развертывание виртуальных машин может потребоваться несколько минут.

[!INCLUDE [ephemeral-ip-note.md](../../includes/ephemeral-ip-note.md)]

## <a name="create-a-public-ip-address---basic"></a>Создание общедоступного IP-адреса, категория "Базовый"

Для доступа к веб-приложению через Интернет подсистеме балансировки нагрузки требуется общедоступный IP-адрес. 

Используйте [az network public-ip create](/cli/azure/network/public-ip#az_network_public_ip_create), чтобы:

* создать стандартный избыточный между зонами общедоступный IP-адрес с именем **myPublicIP**;
* в группе ресурсов **CreatePubLBQS-rg**.

```azurecli-interactive
  az network public-ip create \
    --resource-group CreatePubLBQS-rg \
    --name myPublicIP \
    --sku Basic
```

## <a name="create-basic-load-balancer"></a>Создание подсистемы балансировки нагрузки уровня "Базовый"

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
* связанным с общедоступным IP-адресом **myPublicIP**, созданным на предыдущем шаге. 

```azurecli-interactive
  az network lb create \
    --resource-group CreatePubLBQS-rg \
    --name myLoadBalancer \
    --sku Basic \
    --public-ip-address myPublicIP \
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
    --resource-group CreatePubLBQS-rg \
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

```azurecli-interactive
  az network lb rule create \
    --resource-group CreatePubLBQS-rg \
    --lb-name myLoadBalancer \
    --name myHTTPRule \
    --protocol tcp \
    --frontend-port 80 \
    --backend-port 80 \
    --frontend-ip-name myFrontEnd \
    --backend-pool-name myBackEndPool \
    --probe-name myHealthProbe \
    --idle-timeout 15
```

### <a name="add-virtual-machines-to-load-balancer-backend-pool"></a>Добавление виртуальных машин во внутренний пул подсистемы балансировки нагрузки

Добавьте виртуальные машины во внутренний пул, используя команду [az network nic ip-config address-pool add](/cli/azure/network/nic/ip-config/address-pool#az_network_nic_ip_config_address_pool_add):

* в серверном пуле адресов **myBackEndPool**;
* в группе ресурсов **CreatePubLBQS-rg**;
* связанный с подсистемой балансировки нагрузки **myLoadBalancer**.

```azurecli-interactive
  array=(myNicVM1 myNicVM2 myNicVM3)
  for vmnic in "${array[@]}"
  do
    az network nic ip-config address-pool add \
     --address-pool myBackendPool \
     --ip-config-name ipconfig1 \
     --nic-name $vmnic \
     --resource-group CreatePubLBQS-rg \
     --lb-name myLoadBalancer
  done
```

---

## <a name="install-iis"></a>Установка служб IIS

С помощью [az vm extension set](/cli/azure/vm/extension#az_vm_extension_set) можно установить службы IIS на виртуальных машинах и указать для веб-сайта по умолчанию имя компьютера.

```azurecli-interactive
  array=(myVM1 myVM2 myVM3)
    for vm in "${array[@]}"
    do
     az vm extension set \
       --publisher Microsoft.Compute \
       --version 1.8 \
       --name CustomScriptExtension \
       --vm-name $vm \
       --resource-group CreatePubLBQS-rg \
       --settings '{"commandToExecute":"powershell Add-WindowsFeature Web-Server; powershell Add-Content -Path \"C:\\inetpub\\wwwroot\\Default.htm\" -Value $($env:computername)"}'
  done

```

## <a name="test-the-load-balancer"></a>Тестирование подсистемы балансировки нагрузки

Чтобы получить общедоступный IP-адрес подсистемы балансировки нагрузки, используйте команду [az network public-ip show](/cli/azure/network/public-ip#az_network_public_ip_show). 

Скопируйте общедоступный IP-адрес и вставьте его в адресную строку браузера.

```azurecli-interactive
  az network public-ip show \
    --resource-group CreatePubLBQS-rg \
    --name myPublicIP \
    --query ipAddress \
    --output tsv
```
:::image type="content" source="./media/load-balancer-standard-public-cli/running-nodejs-app.png" alt-text="Тестирование подсистемы балансировки нагрузки" border="true":::

## <a name="clean-up-resources"></a>Очистка ресурсов

Вы можете удалить ненужную группу ресурсов, подсистему балансировки нагрузки и все связанные с ней ресурсы, выполнив команду [az group delete](/cli/azure/group#az_group_delete).

```azurecli-interactive
  az group delete \
    --name CreatePubLBQS-rg
```

## <a name="next-steps"></a>Дальнейшие действия

Из этого краткого руководства вы узнали, как:

* создать стандартную или общедоступную подсистему балансировки нагрузки;
* подключить к ней виртуальные машины; 
* настроить правило трафика подсистемы балансировки нагрузки и пробу работоспособности;
* тестировать подсистему балансировки нагрузки.

Чтобы узнать больше об Azure Load Balancer, ознакомьтесь со следующей статьей:
> [!div class="nextstepaction"]
> [Что такое Azure Load Balancer](load-balancer-overview.md)