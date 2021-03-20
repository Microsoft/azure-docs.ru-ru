---
title: Настройка TCP-сброса и времени ожидания простоя подсистемы балансировки нагрузки
titleSuffix: Azure Load Balancer
description: Из этой статьи вы узнаете, как настроить Azure Load Balancer время ожидания простоя TCP и выполнить сброс.
services: load-balancer
documentationcenter: na
author: asudbring
ms.custom: seodec18
ms.service: load-balancer
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 10/26/2020
ms.author: allensu
ms.openlocfilehash: 8a6be588544883b77c3ff115c9dba5e6ecd5fbd7
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92747200"
---
# <a name="configure-tcp-reset-and-idle-timeout-for-azure-load-balancer"></a>Настройте сброс TCP и время ожидания простоя для Azure Load Balancer

Azure Load Balancer имеет следующий диапазон времени ожидания простоя:

* от 4 минут до 100 минут для исходящих правил
* от 4 до 30 минут для правил Load Balancer и правил NAT для входящего трафика

Значение по умолчанию — 4 минуты. Если период бездействия превышает значение времени ожидания, то нет никакой гарантии, что сеанс TCP или HTTP будет поддерживаться между клиентом и службой. 

В следующих разделах описывается изменение параметров времени ожидания простоя и сброса TCP для ресурсов балансировщика нагрузки.

## <a name="set-tcp-reset-and-idle-timeout"></a>Установка сброса TCP и времени ожидания простоя
---
# <a name="portal"></a>[**Портал**](#tab/tcp-reset-idle-portal)

Чтобы задать время ожидания простоя и сброс TCP для балансировщика нагрузки, измените правило балансировки нагрузки. 

1. Войдите на [портал Azure](https://portal.azure.com).

2. В меню слева выберите **группы ресурсов**.

3. Выберите группу ресурсов для балансировщика нагрузки. В этом примере группа ресурсов называется **myResourceGroup**.

4. Выберите подсистему балансировки нагрузки. В этом примере подсистема балансировки нагрузки называется **myLoadBalancer**.

5. В окне **Параметры** выберите **правила балансировки нагрузки**.

     :::image type="content" source="./media/load-balancer-tcp-idle-timeout/portal-lb-rules.png" alt-text="Изменение правил подсистемы балансировки нагрузки." border="true":::

6. Выберите правило балансировки нагрузки. В этом примере правило балансировки нагрузки называется **myLBrule**.

7. В правиле балансировки нагрузки переместите ползунок **время ожидания простоя (в минутах)** в значение времени ожидания.  

8. В разделе **Сброс TCP** выберите **включено**.

   :::image type="content" source="./media/load-balancer-tcp-idle-timeout/portal-lb-rules-tcp-reset.png" alt-text="Задайте время ожидания простоя и сброс TCP." border="true":::

9. Щелкните **Сохранить**.

# <a name="powershell"></a>[**PowerShell**](#tab/tcp-reset-idle-powershell)

Чтобы установить время ожидания простоя и сброс TCP, задайте значения в следующих параметрах правила балансировки нагрузки с помощью [Set-азлоадбаланцер](/powershell/module/az.network/set-azloadbalancer):

* **идлетимеаутинминутес**
* **енаблеткпресет**

Чтобы установить и использовать PowerShell локально, для работы с этой статьей вам понадобится модуль Azure PowerShell 5.4.1 или более поздней версии. Выполните командлет `Get-Module -ListAvailable Az`, чтобы узнать установленную версию. Если вам необходимо выполнить обновление, ознакомьтесь со статьей, посвященной [установке модуля Azure PowerShell](/powershell/azure/install-Az-ps). При использовании PowerShell на локальном компьютере также нужно запустить `Connect-AzAccount`, чтобы создать подключение к Azure.

Замените следующие примеры значениями из ресурсов:

* **myResourceGroup**
* **myLoadBalancer**

```azurepowershell
$lb = Get-AzLoadBalancer -Name "myLoadBalancer" -ResourceGroup "myResourceGroup"
$lb.LoadBalancingRules[0].IdleTimeoutInMinutes = '15'
$lb.LoadBalancingRules[0].EnableTcpReset = 'true'
Set-AzLoadBalancer -LoadBalancer $lb
```

# <a name="azure-cli"></a>[**Azure CLI**](#tab/tcp-reset-idle-cli)

Чтобы установить время ожидания простоя и сброс TCP, используйте следующие параметры для [AZ Network фунтов правила Update](/cli/azure/network/lb/rule?az_network_lb_rule_update):

* **--время ожидания простоя**
* **--Enable-TCP-Reset**

Перед началом работы проверьте среду, выполнив следующие действия.

* Войдите на портал Azure и убедитесь, что ваша подписка активна, выполнив команду `az login`.
* Проверьте версию Azure CLI в терминале или в командном окне, выполнив команду `az --version`. Сведения о последней версии см. в [заметках о выпуске](/cli/azure/release-notes-azure-cli?tabs=azure-cli).
  * Если у вас нет последней версии, обновите установку, следуя указаниям в [руководстве по установке операционной системы или платформы](/cli/azure/install-azure-cli).

Замените следующие примеры значениями из ресурсов:

* **myResourceGroup**
* **myLoadBalancer**
* **myLBrule**


```azurecli
az network lb rule update \
    --resource-group myResourceGroup \
    --name myLBrule \
    --lb-name myLoadBalancer \
    --idle-timeout 15 \
    --enable-tcp-reset true
```
---
## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о времени ожидания простоя TCP и сбросе см. в разделе [Load Balancer сброс TCP и время ожидания простоя](load-balancer-tcp-reset.md) .

Дополнительные сведения о настройке режима распределения балансировщика нагрузки см. в разделе [Настройка режима распределения балансировщика нагрузки](load-balancer-distribution-mode.md).
