---
title: Учебник по созданию подсистемы балансировки нагрузки между регионами с помощью Azure CLI
titleSuffix: Azure Load Balancer
description: Из этого учебника вы узнаете, как развернуть Azure Load Balancer между регионами с помощью Azure CLI.
author: asudbring
ms.author: allensu
ms.service: load-balancer
ms.topic: tutorial
ms.date: 03/04/2021
ms.openlocfilehash: 83efb428a94d49b77ecd923d4868afe034374b5f
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "103225189"
---
# <a name="tutorial-create-a-cross-region-azure-load-balancer-using-azure-cli"></a>Учебник по созданию Azure Load Balancer между регионами с помощью Azure CLI

Подсистема балансировки нагрузки в нескольких регионах обеспечивает глобальную доступность службы в разных регионах Azure. В случае сбоя одного региона его трафик направляется на подсистему балансировки нагрузки в ближайшем работоспособном регионе.  

В этом руководстве описано следующее.

> [!div class="checklist"]
> * создание подсистемы балансировки нагрузки в нескольких регионах;
> * Создайте правило балансировщика нагрузки.
> * создание серверного пула с двумя региональными подсистемами балансировки нагрузки;
> * тестирование подсистемы балансировки нагрузки.

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

## <a name="prerequisites"></a>Предварительные требования

- Подписка Azure.
- Два Azure Load Balancer ценовой категории **Стандартный**, серверные пулы которых развернуты в двух разных регионах Azure.
    - Сведения о том, как создать региональную подсистему балансировки нагрузки ценовой категории "Стандартный" и виртуальные машины для серверных пулов см. в кратком руководстве [Создание общедоступной подсистемы балансировки нагрузки для распределения нагрузки между виртуальными машинами с помощью Azure CLI](quickstart-load-balancer-standard-public-cli.md).
        - К именам подсистем балансировки нагрузки и виртуальных машин в каждом регионе добавьте **-R1** и **-R2**. 
- Локальная установка Azure CLI или Azure Cloud Shell.

Если вы решили установить и использовать CLI локально, для выполнения инструкций из этого руководства вам потребуется Azure CLI версии 2.0.28 или более поздней версии. Чтобы узнать версию, выполните команду `az --version`. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI]( /cli/azure/install-azure-cli).

## <a name="sign-in-to-azure-cli"></a>Вход в Azure CLI

Войдите в Azure CLI:

```azurecli-interactive
az login
```

## <a name="create-cross-region-load-balancer"></a>Создание подсистемы балансировки нагрузки в нескольких регионах

В этом разделе показано, как создать подсистему балансировки нагрузки в нескольких регионах, общедоступный IP-адрес и правило балансировки нагрузки.

### <a name="create-a-resource-group"></a>Создание группы ресурсов

Группа ресурсов Azure является логическим контейнером, в котором происходит развертывание ресурсов Azure и управление ими.

Создайте группу ресурсов с помощью команды [az group create](/cli/azure/group#az-group-create):

* с именем **myResourceGroupLB-CR**;
* в расположении **westus**.

```azurecli-interactive
  az group create \
    --name myResourceGroupLB-CR \
    --location westus
```

### <a name="create-the-load-balancer-resource"></a>Создание ресурса подсистемы балансировки нагрузки

Создайте подсистему балансировки нагрузки между регионами с помощью командлета [az network cross-region-lb create](/cli/azure/network/cross-region-lb#az_network_cross_region_lb_create):

* с именем **myLoadBalancer-CR**;
* c пулом переднего плана **myFrontEnd-CR**;
* c серверным пулом **myBackEndPool-CR**.

```azurecli-interactive
  az network cross-region-lb create \
    --name myLoadBalancer-CR \
    --resource-group myResourceGroupLB-CR \
    --frontend-ip-name myFrontEnd-CR \
    --backend-pool-name myBackEndPool-CR     
```

### <a name="create-the-load-balancer-rule"></a>Создание правила подсистемы балансировки нагрузки

Правило подсистемы балансировки нагрузки определяет:

* конфигурацию интерфейсных IP-адресов для входящего трафика;
* серверный пул IP-адресов для приема трафика;
* требуемые порты источника и назначения. 

С помощью команды [az network cross-region-lb rule create](/cli/azure/network/cross-region-lb/rule#az_network_cross_region_lb_rule_create) создайте правило подсистемы балансировки нагрузки, которое:

* имеет имя **myHTTPRule-CR**;
* ожидает передачи данных от **порта 80**, используемого интерфейсным пулом **myFrontEnd-CR**;
* перенаправляет трафик, для которого настроена балансировка нагрузки, ко внутреннему пулу адресов **myBackEndPool-CR**, который использует **порт 80**; 
* использует протокол **TCP**;

```azurecli-interactive
  az network cross-region-lb rule create \
    --backend-port 80 \
    --frontend-port 80 \
    --lb-name myLoadBalancer-CR \
    --name myHTTPRule-CR \
    --protocol tcp \
    --resource-group myResourceGroupLB-CR \
    --backend-pool-name myBackEndPool-CR \
    --frontend-ip-name myFrontEnd-CR
```

## <a name="create-backend-pool"></a>Создание внутреннего пула

В этом разделе показано, как добавить в серверный пул две подсистемы балансировки нагрузки ценовой категории "Стандартный".

> [!IMPORTANT]
> Чтобы выполнить эти действия, в подписке должны быть развернуты две региональные подсистемы балансировки нагрузки с серверными пулами.  Дополнительные сведения см. в кратком руководстве **[Создание общедоступной подсистемы балансировки нагрузки для распределения нагрузки между виртуальными машинами с помощью Azure CLI](quickstart-load-balancer-standard-public-cli.md)** .

### <a name="add-the-regional-frontends-to-load-balancer"></a>Добавление внешнего регионального интерфейса в подсистему балансировки нагрузки

Используя инструкции их этого раздела, вы поместите ИД ресурсов двух внешних интерфейсов региональной подсистемы балансировки нагрузки в переменные.  Затем вы будете использовать переменные, чтобы добавить внешние интерфейсы в пул адресов серверной части в подсистеме балансировки нагрузки между регионами.

Получите ИД ресурсов с помощью команды [az network lb frontend-ip show](/cli/azure/network/lb/frontend-ip#az_network_lb_frontend_ip_show).

Выполните команду [az network cross-region-lb address-pool address add](/cli/azure/network/cross-region-lb/address-pool/address#az_network_cross_region_lb_address_pool_address_add), чтобы добавить внешние интерфейсы, которые вы поместили в переменные, в серверный пул подсистемы балансировки нагрузки между регионами.

```azurecli-interactive
  region1id=$(az network lb frontend-ip show \
    --lb-name myLoadBalancer-R1 \
    --name myFrontEnd-R1 \
    --resource-group CreatePubLBQS-rg-r1 \
    --query id \
    --output tsv)

  az network cross-region-lb address-pool address add \
    --frontend-ip-address $region1id \
    --lb-name myLoadBalancer-CR \
    --name myFrontEnd-R1 \
    --pool-name myBackEndPool-CR \
    --resource-group myResourceGroupLB-CR

  region2id=$(az network lb frontend-ip show \
    --lb-name myLoadBalancer-R2 \
    --name myFrontEnd-R2 \
    --resource-group CreatePubLBQS-rg-r2 \
    --query id \
    --output tsv)
  
  az network cross-region-lb address-pool address add \
    --frontend-ip-address $region2id \
    --lb-name myLoadBalancer-CR \
    --name myFrontEnd-R2 \
    --pool-name myBackEndPool-CR \
    --resource-group myResourceGroupLB-CR
```

## <a name="test-the-load-balancer"></a>Тестирование подсистемы балансировки нагрузки

В этом разделе вы проверите подсистему балансировки нагрузки в нескольких регионах. С помощью веб-браузера вы подключитесь к общедоступному IP-адресу.  Вы остановите работу виртуальных машин в одном из серверных пулов подсистемы балансировки нагрузки, а затем отследите выполнение отработки отказа.

1. Чтобы получить общедоступный IP-адрес подсистемы балансировки нагрузки, используйте команду [az network public-ip show](/cli/azure/network/public-ip#az-network-public-ip-show).

    ```azurecli-interactive
      az network public-ip show \
        --resource-group myResourceGroupLB-CR \
        --name PublicIPmyLoadBalancer-CR \
        --query ipAddress \
        --output tsv
    ```
2. Скопируйте общедоступный IP-адрес и вставьте его в адресную строку браузера. В браузере отобразится страница по умолчанию веб-сервера IIS.

3. Остановите все виртуальные машины в серверном пуле одной из региональных подсистем балансировки нагрузки.

4. Обновите страницу в веб-браузере и убедитесь, что отработка отказа в другую региональную подсистему балансировки нагрузки выполнена успешно.

## <a name="clean-up-resources"></a>Очистка ресурсов

Вы можете удалить ненужную группу ресурсов, подсистему балансировки нагрузки и все связанные с ней ресурсы, выполнив команду [az group delete](/cli/azure/group#az-group-delete).

```azurecli-interactive
  az group delete \
    --name myResourceGroupLB-CR
```

## <a name="next-steps"></a>Дальнейшие действия

Изучив это руководство, вы:

* создали подсистему балансировки нагрузки в нескольких регионах;
* создали правило балансировки нагрузки;
* добавили региональные подсистемы балансировки нагрузки в серверный пул подсистемы балансировки нагрузки в нескольких регионах;
* тестировать подсистему балансировки нагрузки.

Дополнительные сведения см. в следующей статье:
> [!div class="nextstepaction"]
> [Распределение нагрузки виртуальных машин в пределах зон доступности](tutorial-load-balancer-standard-public-zone-redundant-portal.md)
