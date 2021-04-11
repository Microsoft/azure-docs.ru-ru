---
title: Краткое руководство. Создание внутренней подсистемы балансировки нагрузки — Azure CLI
titleSuffix: Azure Load Balancer
description: В этом кратком руководстве показано, как создать внутреннюю подсистему балансировки нагрузки с помощью Azure CLI.
services: load-balancer
documentationcenter: na
author: asudbring
manager: KumudD
ms.service: load-balancer
ms.devlang: na
ms.topic: quickstart
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 12/19/2020
ms.author: allensu
ms.custom: mvc, devx-track-js, devx-track-azurecli
ms.openlocfilehash: f728e1f1e2186188135666ed54e02c9ed3507509
ms.sourcegitcommit: 73fb48074c4c91c3511d5bcdffd6e40854fb46e5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/31/2021
ms.locfileid: "106056544"
---
# <a name="quickstart-create-an-internal-load-balancer-by-using-the-azure-cli"></a>Краткое руководство. Создание внутреннего балансировщика нагрузки с помощью Azure CLI

Начните работу с Azure Load Balancer, создав с помощью Azure CLI внутреннюю подсистему балансировки нагрузки и три виртуальные машины.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)] 

Для работы с этим кратким руководством требуется Azure CLI версии 2.0.28 или более поздней. Если вы используете Azure Cloud Shell, последняя версия уже установлена.

>[!NOTE]
>Azure Load Balancer ценовой категории "Стандартный" — это рекомендуемый вариант для рабочих нагрузок в рабочей среде. Эта статья содержит сведения об Azure Load Balancer ценовой категории "Стандартный", а также об Azure Load Balancer ценовой категории "Базовый". Дополнительные сведения о доступных ценовых категориях см. в статье [Номера SKU для Azure Load Balancer](skus.md).

## <a name="create-a-resource-group"></a>Создание группы ресурсов

Группа ресурсов Azure — это логический контейнер, в котором происходит развертывание ресурсов Azure и управление ими.

Создайте группу ресурсов с помощью команды [az group create](/cli/azure/group#az_group_create). Назовите группу ресурсов **CreateIntLBQS-rg** и укажите расположение **eastus**.

```azurecli-interactive
  az group create \
    --name CreateIntLBQS-rg \
    --location eastus

```

## <a name="azure-load-balancer-standard"></a>Azure Load Balancer ценовой категории "Стандартный"

В этом разделе показано, как создать подсистему балансировки нагрузки, которая распределяет нагрузку между виртуальными машинами. При создании внутренней подсистемы балансировки нагрузки виртуальная сеть настраивается в качестве сети для подсистемы балансировки нагрузки. На следующей схеме показаны ресурсы, созданные при работе с этим кратким руководством:

:::image type="content" source="./media/quickstart-load-balancer-standard-internal-portal/resources-diagram-internal.png" alt-text="Ресурсы подсистемы балансировки нагрузки ценовой категории &quot;Стандартный&quot;, созданные для работы с этим кратким руководством." border="false":::

### <a name="configure-the-virtual-network"></a>Настройка виртуальной сети

Прежде чем развертывать виртуальные машины и подсистему балансировки нагрузки, создайте вспомогательные ресурсы виртуальной сети.

#### <a name="create-a-virtual-network"></a>Создание виртуальной сети

Создайте виртуальную сеть с помощью команды [az network vnet create](/cli/azure/network/vnet#az-network-vnet-create). Укажите следующее.

* Имя **myVNet**
* Префикс адреса **10.1.0.0/16**
* Имя подсети **myBackendSubnet**
* Префикс подсети **10.1.0.0/24**
* В группе ресурсов **CreateIntLBQS-rg**
* Расположение **eastus**

```azurecli-interactive
  az network vnet create \
    --resource-group CreateIntLBQS-rg \
    --location eastus \
    --name myVNet \
    --address-prefixes 10.1.0.0/16 \
    --subnet-name myBackendSubnet \
    --subnet-prefixes 10.1.0.0/24
```

#### <a name="create-a-public-ip-address"></a>Создание общедоступного IP-адреса

Чтобы создать общедоступный IP-адрес узла Бастиона Azure, воспользуйтесь командой [az network public-ip create](/cli/azure/network/public-ip#az-network-public-ip-create). Укажите следующее.

* Создайте стандартный избыточный между зонами общедоступный IP-адрес с именем **myPublicIP**.
* в группе ресурсов **CreateIntLBQS-rg**;

```azurecli-interactive
az network public-ip create \
    --resource-group CreateIntLBQS-rg  \
    --name myBastionIP \
    --sku Standard
```
#### <a name="create-an-azure-bastion-subnet"></a>Создание подсети Бастиона Azure

Используйте команду [az network vnet subnet create](/cli/azure/network/vnet/subnet#az-network-vnet-subnet-create), чтобы создать подсеть. Укажите следующее.

* Имя **AzureBastionSubnet**
* Префикс адреса **10.1.1.0/24**
* Виртуальная сеть **myVNet**
* В группе ресурсов **CreateIntLBQS-rg**

```azurecli-interactive
az network vnet subnet create \
    --resource-group CreateIntLBQS-rg  \
    --name AzureBastionSubnet \
    --vnet-name myVNet \
    --address-prefixes 10.1.1.0/24
```

#### <a name="create-an-azure-bastion-host"></a>Создание узла-бастиона Azure

Используйте команду [az network bastion create](/cli/azure/network/bastion#az-network-bastion-create) для создания узла. Укажите следующее.

* с именем **myBastionHost**;
* в группе ресурсов **CreateIntLBQS-rg**;
* Связанный с общедоступным IP-адресом **myBastionIP**
* Связанный с виртуальной сетью **myVNet**
* В расположении **eastus**

```azurecli-interactive
az network bastion create \
    --resource-group CreateIntLBQS-rg  \
    --name myBastionHost \
    --public-ip-address myBastionIP \
    --vnet-name myVNet \
    --location eastus
```

Развертывание узла-бастиона может занять несколько минут.

#### <a name="create-a-network-security-group"></a>Создание группы безопасности сети

Для стандартной подсистемы балансировки нагрузки убедитесь, что сетевые интерфейсы ваших виртуальных машин принадлежат группе безопасности сети. Создайте группу безопасности сети с помощью команды [az network nsg create](/cli/azure/network/nsg#az-network-nsg-create). Укажите следующее.

* Имя **myNSG**
* В группе ресурсов **CreateIntLBQS-rg**

```azurecli-interactive
  az network nsg create \
    --resource-group CreateIntLBQS-rg \
    --name myNSG
```

#### <a name="create-a-network-security-group-rule"></a>Создание правила группы безопасности сети

Создайте правило группы безопасности сети с помощью команды [az network nsg rule create](/cli/azure/network/nsg/rule#az-network-nsg-rule-create). Укажите следующее.

* Имя **myNSGRuleHTTP**
* В группе безопасности сети, созданной на предыдущем шаге, **myNSG**
* В группе ресурсов **CreateIntLBQS-rg**
* Протокол **(*)**
* Направление **Входящий**
* Источник **(*)**
* Назначение **(*)**
* Порт назначения **Порт 80**
* Доступ **Разрешить**
* Приоритет **200**

```azurecli-interactive
  az network nsg rule create \
    --resource-group CreateIntLBQS-rg \
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

### <a name="create-back-end-servers"></a>Создание внутренних серверов

В этом разделе показано, как создать:

* три сетевых интерфейса для виртуальных машин;
* три виртуальные машины, которые будут использоваться в качестве серверов для подсистемы балансировки нагрузки.

#### <a name="create-network-interfaces-for-the-virtual-machines"></a>Создание сетевых интерфейсов для виртуальных машин

Создайте три сетевых интерфейса с помощью команды [az network nic create](/cli/azure/network/nic#az-network-nic-create). Укажите следующее.

* Имена **myNicVM1**, **myNicVM2** и **myNicVM3**
* В группе ресурсов **CreateIntLBQS-rg**
* Виртуальная сеть **myVNet**
* В подсети **myBackendSubnet**
* В группе безопасности сети **myNSG**

```azurecli-interactive
  array=(myNicVM1 myNicVM2 myNicVM3)
  for vmnic in "${array[@]}"
  do
    az network nic create \
        --resource-group CreateIntLBQS-rg \
        --name $vmnic \
        --vnet-name myVNet \
        --subnet myBackEndSubnet \
        --network-security-group myNSG
  done
```

#### <a name="create-the-virtual-machines"></a>Создание виртуальных машин

Создайте виртуальные машины с помощью команды [az vm create](/cli/azure/vm#az-vm-create). Укажите следующее.

* Имена **myVM1**, **myVM2** и **myVM3**
* В группе ресурсов **CreateIntLBQS-rg**
* С подключением к сетевым интерфейсам **myNicVM1**, **myNicVM2** и **myNicVM3**
* Образ виртуальной машины **win2019datacenter**
* В зонах **Zone 1**, **Zone 2** и **Zone 3**

```azurecli-interactive
  array=(1 2 3)
  for n in "${array[@]}"
  do
    az vm create \
    --resource-group CreateIntLBQS-rg \
    --name myVM$n \
    --nics myNicVM$n \
    --image win2019datacenter \
    --admin-username azureuser \
    --zone $n \
    --no-wait
  done
```

На развертывание виртуальных машин может потребоваться несколько минут.

### <a name="create-the-load-balancer"></a>Создание подсистемы балансировки нагрузки

В этом разделе описано, как создать и настроить следующие компоненты подсистемы балансировки нагрузки:

* пул IP-адресов, который получает входящий трафик в подсистеме балансировки нагрузки;
* второй пул IP-адресов, на который первый пул отправляет трафик с балансировкой нагрузки;
* проверка работоспособности, определяющая работоспособность экземпляров виртуальной машины;
* правило подсистемы балансировки нагрузки, определяющее порядок распределения трафика между виртуальными машинами.

#### <a name="create-the-load-balancer-resource"></a>Создание ресурса подсистемы балансировки нагрузки

С помощью команды [az network lb create](/cli/azure/network/lb#az-network-lb-create) создайте общедоступную подсистему балансировки нагрузки. Укажите следующее.

* с именем **myLoadBalancer**;
* Имя пула **myFrontEnd**
* Имя пула **myBackEndPool**
* Связанная с виртуальной сетью **myVNet**
* Связанная с внутренней подсетью **myBackendSubnet**

```azurecli-interactive
  az network lb create \
    --resource-group CreateIntLBQS-rg \
    --name myLoadBalancer \
    --sku Standard \
    --vnet-name myVnet \
    --subnet myBackendSubnet \
    --frontend-ip-name myFrontEnd \
    --backend-pool-name myBackEndPool
```

#### <a name="create-the-health-probe"></a>Создание зонда работоспособности

При пробе работоспособности выполняется проверка всех экземпляров виртуальной машины, чтобы убедиться, что они могут отправлять сетевой трафик. Виртуальная машина с неудачной пробой удаляется из подсистемы балансировки нагрузки и снова добавляется в нее после устранения сбоя.

Создайте пробу работоспособности с помощью команды [az network lb probe create](/cli/azure/network/lb/probe#az-network-lb-probe-create). Укажите следующее.

* Отслеживание работоспособности виртуальных машин
* Имя **myHealthProbe**
* Протокол **TCP**
* Отслеживает **Порт 80**

```azurecli-interactive
  az network lb probe create \
    --resource-group CreateIntLBQS-rg \
    --lb-name myLoadBalancer \
    --name myHealthProbe \
    --protocol tcp \
    --port 80
```

#### <a name="create-a-load-balancer-rule"></a>Создание правила балансировщика нагрузки

Правило подсистемы балансировки нагрузки определяет:

* конфигурацию IP-адресов для входящего трафика;
* пул IP-адресов для получения трафика;
* требуемые порты источника и назначения. 

Создайте правило балансировщика нагрузки с помощью команды [az network lb rule create](/cli/azure/network/lb/rule#az-network-lb-rule-create). Укажите следующее.

* имеет имя **myHTTPRule**;
* Прослушивание **Порт 80** в пуле **myFrontEnd**
* Перенаправление трафика, для которого настроена балансировка нагрузки, к пулу адресов **myBackEndPool**, который использует **Порт 80** 
* Использование пробы работоспособности **myHealthProbe**
* Протокол **TCP**
* Время ожидания перед переходом в режим простоя: **15 минут**
* Включить функцию сброса подключений TCP

```azurecli-interactive
  az network lb rule create \
    --resource-group CreateIntLBQS-rg \
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

#### <a name="add-vms-to-the-load-balancer-pool"></a>Добавить виртуальные машины в подсистему балансировки нагрузки

Добавьте виртуальные машины во внутренний пул, используя команду [az network nic ip-config address-pool add](/cli/azure/network/nic/ip-config/address-pool#az-network-nic-ip-config-address-pool-add). Укажите следующее.

* В пуле адресов **myBackEndPool**
* В группе ресурсов **CreateIntLBQS-rg**
* С привязкой к сетевым интерфейсам **myNicVM1**, **myNicVM2** и **myNicVM3**
* связь с подсистемой балансировки нагрузки **myLoadBalancer**;

```azurecli-interactive
  array=(VM1 VM2 VM3)
  for vm in "${array[@]}"
  do
  az network nic ip-config address-pool add \
   --address-pool myBackendPool \
   --ip-config-name ipconfig1 \
   --nic-name myNic$vm \
   --resource-group CreateIntLBQS-rg \
   --lb-name myLoadBalancer
  done

```

## <a name="azure-load-balancer-basic"></a>Azure Load Balancer ценовой категории "Базовый"

В этом разделе показано, как создать подсистему балансировки нагрузки, которая распределяет нагрузку между виртуальными машинами. При создании внутренней подсистемы балансировки нагрузки виртуальная сеть настраивается в качестве сети для подсистемы балансировки нагрузки. На следующей схеме показаны ресурсы, созданные при работе с этим кратким руководством:

:::image type="content" source="./media/quickstart-load-balancer-standard-internal-portal/resources-diagram-internal-basic.png" alt-text="Ресурсы подсистемы балансировки нагрузки ценовой категории &quot;Базовый&quot;, созданные для работы с этим кратким руководством." border="false":::

### <a name="configure-the-virtual-network"></a>Настройка виртуальной сети

Прежде чем развертывать виртуальные машины и подсистему балансировки нагрузки, создайте вспомогательные ресурсы виртуальной сети.

#### <a name="create-a-virtual-network"></a>Создание виртуальной сети

Создайте виртуальную сеть с помощью команды [az network vnet create](/cli/azure/network/vnet#az-network-vnet-createt). Укажите следующее.

* Имя **myVNet**
* Префикс адреса **10.1.0.0/16**
* Имя подсети **myBackendSubnet**
* Префикс подсети **10.1.0.0/24**
* В группе ресурсов **CreateIntLBQS-rg**
* Расположение **eastus**

```azurecli-interactive
  az network vnet create \
    --resource-group CreateIntLBQS-rg \
    --location eastus \
    --name myVNet \
    --address-prefixes 10.1.0.0/16 \
    --subnet-name myBackendSubnet \
    --subnet-prefixes 10.1.0.0/24
```

#### <a name="create-a-public-ip-address"></a>Создание общедоступного IP-адреса

Чтобы создать общедоступный IP-адрес узла Бастиона Azure, воспользуйтесь командой [az network public-ip create](/cli/azure/network/public-ip#az-network-public-ip-create). Укажите следующее.

* Создайте стандартный избыточный между зонами общедоступный IP-адрес с именем **myPublicIP**.
* в группе ресурсов **CreateIntLBQS-rg**;

```azurecli-interactive
az network public-ip create \
    --resource-group CreateIntLBQS-rg \
    --name myBastionIP \
    --sku Standard
```
#### <a name="create-an-azure-bastion-subnet"></a>Создание подсети Бастиона Azure

Используйте команду [az network vnet subnet create](/cli/azure/network/vnet/subnet#az-network-vnet-subnet-create), чтобы создать подсеть. Укажите следующее.

* Имя **AzureBastionSubnet**
* Префикс адреса **10.1.1.0/24**
* Виртуальная сеть **myVNet**
* В группе ресурсов **CreateIntLBQS-rg**

```azurecli-interactive
az network vnet subnet create \
    --resource-group CreateIntLBQS-rg \
    --name AzureBastionSubnet \
    --vnet-name myVNet \
    --address-prefixes 10.1.1.0/24
```

#### <a name="create-an-azure-bastion-host"></a>Создание узла-бастиона Azure

Используйте команду [az network bastion create](/cli/azure/network/bastion#az-network-bastion-create) для создания узла. Укажите следующее.

* с именем **myBastionHost**;
* в группе ресурсов **CreateIntLBQS-rg**;
* Связанный с общедоступным IP-адресом **myBastionIP**
* Связанный с виртуальной сетью **myVNet**
* В расположении **eastus**

```azurecli-interactive
az network bastion create \
    --resource-group CreateIntLBQS-rg \
    --name myBastionHost \
    --public-ip-address myBastionIP \
    --vnet-name myVNet \
    --location eastus
```

Развертывание узла-бастиона может занять несколько минут.

#### <a name="create-a-network-security-group"></a>Создание группы безопасности сети

Для стандартной подсистемы балансировки нагрузки убедитесь, что сетевые интерфейсы ваших виртуальных машин принадлежат группе безопасности сети. Создайте группу безопасности сети с помощью команды [az network nsg create](/cli/azure/network/nsg#az-network-nsg-create). Укажите следующее.

* Имя **myNSG**
* В группе ресурсов **CreateIntLBQS-rg**

```azurecli-interactive
  az network nsg create \
    --resource-group CreateIntLBQS-rg \
    --name myNSG
```

#### <a name="create-a-network-security-group-rule"></a>Создание правила группы безопасности сети

Создайте правило группы безопасности сети с помощью команды [az network nsg rule create](/cli/azure/network/nsg/rule#az-network-nsg-rule-create). Укажите следующее.

* Имя **myNSGRuleHTTP**
* В группе безопасности сети, созданной на предыдущем шаге, **myNSG**
* В группе ресурсов **CreateIntLBQS-rg**
* Протокол **(*)**
* Направление **Входящий**
* Источник **(*)**
* Назначение **(*)**
* Порт назначения **Порт 80**
* Доступ **Разрешить**
* Приоритет **200**

```azurecli-interactive
  az network nsg rule create \
    --resource-group CreateIntLBQS-rg \
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

### <a name="create-back-end-servers"></a>Создание внутренних серверов

В этом разделе показано, как создать:

* три сетевых интерфейса для виртуальных машин;
* группу доступности для виртуальных машин;
* три виртуальные машины, которые будут использоваться в качестве серверов для подсистемы балансировки нагрузки.

#### <a name="create-network-interfaces-for-the-virtual-machines"></a>Создание сетевых интерфейсов для виртуальных машин

Создайте три сетевых интерфейса с помощью команды [az network nic create](/cli/azure/network/nic#az-network-nic-create). Укажите следующее.

* Имена **myNicVM1**, **myNicVM2** и **myNicVM3**
* В группе ресурсов **CreateIntLBQS-rg**
* Виртуальная сеть **myVNet**
* В подсети **myBackendSubnet**
* В группе безопасности сети **myNSG**

```azurecli-interactive
  array=(myNicVM1 myNicVM2 myNicVM3)
  for vmnic in "${array[@]}"
  do
    az network nic create \
        --resource-group CreateIntLBQS-rg \
        --name $vmnic \
        --vnet-name myVNet \
        --subnet myBackEndSubnet \
        --network-security-group myNSG
  done
```

#### <a name="create-the-availability-set-for-the-virtual-machines"></a>Создание группы доступности для виртуальных машин

Создайте группу доступности с помощью команды [az vm availability-set create](/cli/azure/vm/availability-set#az-vm-availability-set-create). Укажите следующее.

* Имя **myAvailabilitySet**
* В группе ресурсов **CreateIntLBQS-rg**
* Расположение **eastus**

```azurecli-interactive
  az vm availability-set create \
    --name myAvailabilitySet \
    --resource-group CreateIntLBQS-rg \
    --location eastus 
    
```

#### <a name="create-the-virtual-machines"></a>Создание виртуальных машин

Создайте виртуальные машины с помощью команды [az vm create](/cli/azure/vm#az-vm-create). Укажите следующее.

* Имена **myVM1**, **myVM2** и **myVM3**
* В группе ресурсов **CreateIntLBQS-rg**
* С подключением к сетевым интерфейсам **myNicVM1**, **myNicVM2** и **myNicVM3**
* Образ виртуальной машины **win2019datacenter**
* В **myAvailabilitySet**


```azurecli-interactive
  array=(1 2 3)
  for n in "${array[@]}"
  do
    az vm create \
    --resource-group CreateIntLBQS-rg \
    --name myVM$n \
    --nics myNicVM$n \
    --image win2019datacenter \
    --admin-username azureuser \
    --availability-set myAvailabilitySet \
    --no-wait
  done
```
На развертывание виртуальных машин может потребоваться несколько минут.

### <a name="create-the-load-balancer"></a>Создание подсистемы балансировки нагрузки

В этом разделе описано, как создать и настроить следующие компоненты подсистемы балансировки нагрузки:

* пул IP-адресов, который получает входящий трафик в подсистеме балансировки нагрузки;
* второй пул IP-адресов, на который первый пул отправляет трафик с балансировкой нагрузки;
* проверка работоспособности, определяющая работоспособность экземпляров виртуальной машины;
* правило подсистемы балансировки нагрузки, определяющее порядок распределения трафика между виртуальными машинами.

#### <a name="create-the-load-balancer-resource"></a>Создание ресурса подсистемы балансировки нагрузки

С помощью команды [az network lb create](/cli/azure/network/lb#az-network-lb-create) создайте общедоступную подсистему балансировки нагрузки. Укажите следующее.

* с именем **myLoadBalancer**;
* Имя пула **myFrontEnd**
* Имя пула **myBackEndPool**
* Связанная с виртуальной сетью **myVNet**
* Связанная с внутренней подсетью **myBackendSubnet**

```azurecli-interactive
  az network lb create \
    --resource-group CreateIntLBQS-rg \
    --name myLoadBalancer \
    --sku Basic \
    --vnet-name myVNet \
    --subnet myBackendSubnet \
    --frontend-ip-name myFrontEnd \
    --backend-pool-name myBackEndPool
```

#### <a name="create-the-health-probe"></a>Создание зонда работоспособности

При пробе работоспособности выполняется проверка всех экземпляров виртуальной машины, чтобы убедиться, что они могут отправлять сетевой трафик. Виртуальная машина с неудачной пробой удаляется из подсистемы балансировки нагрузки и снова добавляется в нее после устранения сбоя.

Создайте пробу работоспособности с помощью команды [az network lb probe create](/cli/azure/network/lb/probe#az-network-lb-probe-create). Укажите следующее.

* Отслеживание работоспособности виртуальных машин
* Имя **myHealthProbe**
* Протокол **TCP**
* Отслеживает **Порт 80**

```azurecli-interactive
  az network lb probe create \
    --resource-group CreateIntLBQS-rg \
    --lb-name myLoadBalancer \
    --name myHealthProbe \
    --protocol tcp \
    --port 80
```

#### <a name="create-a-load-balancer-rule"></a>Создание правила балансировщика нагрузки

Правило подсистемы балансировки нагрузки определяет:

* конфигурацию IP-адресов для входящего трафика;
* пул IP-адресов для получения трафика;
* требуемые порты источника и назначения. 

Создайте правило балансировщика нагрузки с помощью команды [az network lb rule create](/cli/azure/network/lb/rule#az-network-lb-rule-create). Укажите следующее.

* имеет имя **myHTTPRule**;
* Прослушивание **Порт 80** в пуле **myFrontEnd**
* Перенаправление трафика, для которого настроена балансировка нагрузки, к пулу адресов **myBackEndPool**, который использует **Порт 80** 
* Использование пробы работоспособности **myHealthProbe**
* Протокол **TCP**
* Время ожидания перед переходом в режим простоя: **15 минут**

```azurecli-interactive
  az network lb rule create \
    --resource-group CreateIntLBQS-rg \
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
#### <a name="add-vms-to-the-load-balancer-pool"></a>Добавить виртуальные машины в подсистему балансировки нагрузки

Добавьте виртуальные машины во внутренний пул, используя команду [az network nic ip-config address-pool add](/cli/azure/network/nic/ip-config/address-pool#az-network-nic-ip-config-address-pool-add). Укажите следующее.

* В пуле адресов **myBackEndPool**
* В группе ресурсов **CreateIntLBQS-rg**
* С привязкой к сетевым интерфейсам **myNicVM1**, **myNicVM2** и **myNicVM3**
* связь с подсистемой балансировки нагрузки **myLoadBalancer**;

```azurecli-interactive
  array=(VM1 VM2 VM3)
  for vm in "${array[@]}"
  do
  az network nic ip-config address-pool add \
   --address-pool myBackendPool \
   --ip-config-name ipconfig1 \
   --nic-name myNic$vm \
   --resource-group CreateIntLBQS-rg \
   --lb-name myLoadBalancer
  done

```

## <a name="test-the-load-balancer"></a>Тестирование подсистемы балансировки нагрузки

Создайте сетевой интерфейс с помощью команды [az network nic create](/cli/azure/network/nic#az-network-nic-create). Укажите следующее.

* Имя **myNicTestVM**
* В группе ресурсов **CreateIntLBQS-rg**
* Виртуальная сеть **myVNet**
* В подсети **myBackendSubnet**
* В группе безопасности сети **myNSG**

```azurecli-interactive
  az network nic create \
    --resource-group CreateIntLBQS-rg \
    --name myNicTestVM \
    --vnet-name myVNet \
    --subnet myBackEndSubnet \
    --network-security-group myNSG
```
Создайте виртуальную машину с помощью команды [az vm create](/cli/azure/vm#az-vm-create). Укажите следующее.

* Имя **myTestVM**
* В группе ресурсов **CreateIntLBQS-rg**
* С подключением к сетевому интерфейсу **myNicTestVM**
* Образ виртуальной машины **Win2019Datacenter**

```azurecli-interactive
  az vm create \
    --resource-group CreateIntLBQS-rg \
    --name myTestVM \
    --nics myNicTestVM \
    --image Win2019Datacenter \
    --admin-username azureuser \
    --no-wait
```
Возможно, вам придется подождать несколько минут, пока виртуальная машина развернется.

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
       --resource-group CreateIntLBQS-rg \
       --settings '{"commandToExecute":"powershell Add-WindowsFeature Web-Server; powershell Add-Content -Path \"C:\\inetpub\\wwwroot\\Default.htm\" -Value $($env:computername)"}'
  done

```

### <a name="test"></a>Тест

1. [Войдите](https://portal.azure.com) на портал Azure.

2. Найдите частный IP-адрес для подсистемы балансировки нагрузки на странице **Обзор**. В меню слева выберите **Все службы** > **Все ресурсы** > **myLoadBalancer**.

3. Скопируйте адрес рядом с **частным IP-адресом** на странице обзора **myLoadBalancer**.

4. В меню слева выберите **Все службы** > **Все ресурсы**. В списке ресурсов в группе ресурсов **CreateIntLBQS-rg** выберите **myTestVM**.

5. На странице **Обзор** выберите **Подключиться** > **Бастион**.

6. Введите имя пользователя и пароль, которые вы ввели при создании виртуальной машины.

7. На виртуальной машине **myTestVM** откройте браузер **Internet Explorer**.

8. Введите IP-адрес с предыдущего шага в адресную строку браузера. В браузере отображается страница веб-сервера IIS по умолчанию.

    :::image type="content" source="./media/quickstart-load-balancer-standard-internal-portal/load-balancer-test.png" alt-text="Снимок экрана с IP-адресом в адресной строке браузера." border="true":::
   
Чтобы увидеть, как подсистема балансировки нагрузки распределяет трафик между всеми тремя виртуальными машинами, вы можете настроить для каждого веб-сервера IIS виртуальной машины страницу по умолчанию. Затем вручную обновите веб-браузер с клиентского компьютера.

## <a name="clean-up-resources"></a>Очистка ресурсов

Когда ресурсы больше не нужны, можно удалить группу ресурсов, подсистему балансировки нагрузки и все связанные с ней ресурсы, выполнив команду [az group delete](/cli/azure/group#az-group-delete).

```azurecli-interactive
  az group delete \
    --name CreateIntLBQS-rg
```

## <a name="next-steps"></a>Дальнейшие действия

Ознакомление с общими сведениями об Azure Load Balancer.
> [!div class="nextstepaction"]
> [Что такое Azure Load Balancer](load-balancer-overview.md)
