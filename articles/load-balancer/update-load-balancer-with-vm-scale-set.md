---
title: Обновление или удаление существующей подсистемы балансировки нагрузки, используемой масштабируемыми наборами виртуальных машин
titleSuffix: Update or delete an existing load balancer used by virtual machine scale sets
description: В этом пошаговом руководстве приступите к работе с Azure Load Balancer (цен. категория "Стандартный") и масштабируемыми наборами виртуальных машин.
services: load-balancer
documentationcenter: na
author: irenehua
ms.custom: seodec18
ms.service: load-balancer
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 12/29/2020
ms.author: irenehua
ms.openlocfilehash: 52f2a2ed301bf734ad605a2ee68a0ab672a97014
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102218729"
---
# <a name="update-or-delete-a-load-balancer-used-by-virtual-machine-scale-sets"></a>Обновление или удаление подсистемы балансировки нагрузки, используемой масштабируемыми наборами виртуальных машин

При работе с масштабируемыми наборами виртуальных машин и экземпляром Azure Load Balancer можно:

- Добавление, обновление и удаление правил.
- Добавление конфигураций.
- Удалите подсистему балансировки нагрузки.

## <a name="set-up-a-load-balancer-for-scaling-out-virtual-machine-scale-sets"></a>Настройка подсистемы балансировки нагрузки для масштабирования масштабируемых наборов виртуальных машин

Убедитесь, что в экземпляре Azure Load Balancer настроен [входящий пул NAT](/cli/azure/network/lb/inbound-nat-pool) , а масштабируемый набор виртуальных машин размещен во внутреннем пуле балансировщика нагрузки. Load Balancer автоматически создаст новые правила NAT для входящего трафика в пуле входящих подключений NAT при добавлении новых экземпляров виртуальных машин в масштабируемый набор виртуальных машин.

Чтобы проверить, правильно ли настроен входящий пул NAT, выполните следующие действия.

1. Войдите на [портал Azure](https://portal.azure.com).
1. В меню слева выберите **Все ресурсы**. Затем выберите **MyLoadBalancer** в списке ресурсов.
1. В разделе **Параметры** выберите **правила NAT для входящего трафика**. Если на панели справа отображается список правил, созданных для каждого отдельного экземпляра в масштабируемом наборе виртуальных машин, все готово к масштабированию в любое время.

## <a name="add-inbound-nat-rules"></a>Добавление правил NAT для входящего трафика

Невозможно добавить индивидуальные правила NAT для входящего трафика. Но вы можете добавить набор правил NAT для входящего трафика с определенным диапазоном портов переднего плана и внутренним портом для всех экземпляров в масштабируемом наборе виртуальных машин.

Чтобы добавить весь набор правил NAT для входящего трафика для масштабируемых наборов виртуальных машин, сначала создайте пул входящих подключений NAT в подсистеме балансировки нагрузки. Затем укажите ссылку на пул NAT для входящего трафика из сетевого профиля масштабируемого набора виртуальных машин. Ниже показан полный пример использования интерфейса командной строки.

Новый пул NAT для входящего трафика не должен иметь перекрывающийся диапазон портов внешнего интерфейса с существующими пулами входящего трафика NAT. Чтобы просмотреть настроенные пулы NAT для входящего трафика, используйте следующую [команду CLI](/cli/azure/network/lb/inbound-nat-pool#az_network_lb_inbound_nat_pool_list):
  
```azurecli-interactive
  az network lb inbound-nat-pool create 
          -g MyResourceGroup 
          --lb-name MyLb
          -n MyNatPool 
          --protocol Tcp 
          --frontend-port-range-start 80 
          --frontend-port-range-end 89 
          --backend-port 80 
          --frontend-ip-name MyFrontendIp
  az vmss update 
          -g MyResourceGroup 
          -n myVMSS 
          --add virtualMachineProfile.networkProfile.networkInterfaceConfigurations[0].ipConfigurations[0].loadBalancerInboundNatPools "{'id':'/subscriptions/mySubscriptionId/resourceGroups/MyResourceGroup/providers/Microsoft.Network/loadBalancers/MyLb/inboundNatPools/MyNatPool'}"
            
  az vmss update-instances
          -–instance-ids *
          --resource-group MyResourceGroup
          --name MyVMSS
```
## <a name="update-inbound-nat-rules"></a>Обновление правил NAT для входящего трафика

Невозможно обновить индивидуальные правила NAT для входящего трафика. Но вы можете обновить набор правил NAT для входящего трафика с определенным диапазоном портов внешнего интерфейса и внутренним портом для всех экземпляров в масштабируемом наборе виртуальных машин.

Чтобы обновить весь набор правил NAT для входящего трафика для масштабируемых наборов виртуальных машин, обновите пул NAT для входящего трафика в подсистеме балансировки нагрузки.
    
```azurecli-interactive
az network lb inbound-nat-pool update 
        -g MyResourceGroup 
        --lb-name MyLb 
        -n MyNatPool
        --protocol Tcp 
        --backend-port 8080
```

## <a name="delete-inbound-nat-rules"></a>Удаление правил NAT для входящего трафика

Индивидуальные правила NAT для входящего трафика не могут быть удалены, но можно удалить весь набор правил NAT для входящего трафика, удалив пул входящих подключений NAT.

Чтобы удалить пул NAT, сначала удалите его из масштабируемого набора. Полный пример использования CLI показан здесь:

```azurecli-interactive
    az vmss update
       --resource-group MyResourceGroup
       --name MyVMSS
       --remove virtualMachineProfile.networkProfile.networkInterfaceConfigurations[0].ipConfigurations[0].loadBalancerInboundNatPools
     az vmss update-instances 
       --instance-ids "*" 
       --resource-group MyResourceGroup
       --name MyVMSS
    az network lb inbound-nat-pool delete
       --resource-group MyResourceGroup
       -–lb-name MyLoadBalancer
       --name MyNatPool
```

## <a name="add-multiple-ip-configurations"></a>Добавление нескольких IP-конфигураций

Чтобы добавить несколько IP-конфигураций:

1. В меню слева выберите **Все ресурсы**. Затем выберите **MyLoadBalancer** в списке ресурсов.
1. В разделе **Параметры** выберите **Интерфейсная IP-конфигурация**. Нажмите кнопку **Добавить**.
1. На странице **Добавление внешнего IP-адреса** введите значения и нажмите кнопку **ОК**.
1. Выполните [шаги 5](./load-balancer-multiple-ip.md#step-5-configure-the-health-probe) и [6](./load-balancer-multiple-ip.md#step-5-configure-the-health-probe) этого руководства, если требуются новые правила балансировки нагрузки.
1. При необходимости создайте новый набор правил NAT для входящего трафика с использованием вновь созданных IP-конфигураций внешнего интерфейса. Пример приведен в предыдущем разделе.

## <a name="multiple-virtual-machine-scale-sets-behind-a-single-load-balancer"></a>Несколько масштабируемых наборов виртуальных машин за один Load Balancer

Создайте пул NAT для входящего трафика в Load Balancer, сослаться на пул NAT для входящего трафика в сетевом профиле масштабируемого набора виртуальных машин и, наконец, обновите экземпляры, чтобы изменения вступили в силу. Повторите шаги для всех масштабируемых наборов виртуальных машин.

Не забудьте создать отдельные пулы NAT для входящего трафика с непересекающимися диапазонами интерфейсных портов.
  
```azurecli-interactive
  az network lb inbound-nat-pool create 
          -g MyResourceGroup 
          --lb-name MyLb
          -n MyNatPool 
          --protocol Tcp 
          --frontend-port-range-start 80 
          --frontend-port-range-end 89 
          --backend-port 80 
          --frontend-ip-name MyFrontendIpConfig
  az vmss update 
          -g MyResourceGroup 
          -n myVMSS 
          --add virtualMachineProfile.networkProfile.networkInterfaceConfigurations[0].ipConfigurations[0].loadBalancerInboundNatPools "{'id':'/subscriptions/mySubscriptionId/resourceGroups/MyResourceGroup/providers/Microsoft.Network/loadBalancers/MyLb/inboundNatPools/MyNatPool'}"
            
  az vmss update-instances
          -–instance-ids *
          --resource-group MyResourceGroup
          --name MyVMSS
          
  az network lb inbound-nat-pool create 
          -g MyResourceGroup 
          --lb-name MyLb
          -n MyNatPool2
          --protocol Tcp 
          --frontend-port-range-start 100 
          --frontend-port-range-end 109 
          --backend-port 80 
          --frontend-ip-name MyFrontendIpConfig2
  az vmss update 
          -g MyResourceGroup 
          -n myVMSS2 
          --add virtualMachineProfile.networkProfile.networkInterfaceConfigurations[0].ipConfigurations[0].loadBalancerInboundNatPools "{'id':'/subscriptions/mySubscriptionId/resourceGroups/MyResourceGroup/providers/Microsoft.Network/loadBalancers/MyLb/inboundNatPools/MyNatPool2'}"
            
  az vmss update-instances
          -–instance-ids *
          --resource-group MyResourceGroup
          --name MyVMSS2
```

## <a name="delete-the-front-end-ip-configuration-used-by-the-virtual-machine-scale-set"></a>Удаление IP-конфигурации внешнего интерфейса, используемой масштабируемым набором виртуальных машин

Чтобы удалить конфигурацию внешнего IP-адреса, используемую масштабируемым набором, выполните следующие действия.

 1. Сначала удалите пул NAT для входящего трафика (набор правил NAT для входящего трафика), который ссылается на конфигурацию IP-адресов внешнего интерфейса. Инструкции по удалению правил для входящих подключений содержатся в предыдущем разделе.
 1. Удалите правило балансировки нагрузки, которое ссылается на конфигурацию IP-адресов интерфейсной части.
 1. Удаление IP-конфигурации внешнего интерфейса.

## <a name="delete-a-load-balancer-used-by-a-virtual-machine-scale-set"></a>Удаление подсистемы балансировки нагрузки, используемой масштабируемым набором виртуальных машин

Чтобы удалить конфигурацию внешнего IP-адреса, используемую масштабируемым набором, выполните следующие действия.

 1. Сначала удалите пул NAT для входящего трафика (набор правил NAT для входящего трафика), который ссылается на конфигурацию IP-адресов внешнего интерфейса. Инструкции по удалению правил для входящих подключений содержатся в предыдущем разделе.
 1. Удалите правило балансировки нагрузки, которое ссылается на серверный пул, содержащий масштабируемый набор виртуальных машин.
 1. Удалите `loadBalancerBackendAddressPool` ссылку из сетевого профиля масштабируемого набора виртуальных машин.
 
 Полный пример использования CLI показан здесь:

```azurecli-interactive
    az vmss update
       --resource-group MyResourceGroup
       --name MyVMSS
       --remove virtualMachineProfile.networkProfile.networkInterfaceConfigurations[0].ipConfigurations[0].loadBalancerBackendAddressPools
    az vmss update-instances 
       --instance-ids "*" 
       --resource-group MyResourceGroup
       --name MyVMSS
```
Наконец, удалите ресурс балансировщика нагрузки.
 
## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о Azure Load Balancer и масштабируемых наборах виртуальных машин см. в статье о концепциях.

> [Azure Load Balancer с масштабируемыми наборами виртуальных машин](load-balancer-standard-virtual-machine-scale-sets.md)
