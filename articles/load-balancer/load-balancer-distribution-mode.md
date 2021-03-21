---
title: Настройка режима распределения Azure Load Balancer
titleSuffix: Azure Load Balancer
description: В этой статье приступите к настройке режима распределения для Azure Load Balancer для поддержки сопоставления исходного IP-адреса.
services: load-balancer
documentationcenter: na
author: asudbring
ms.service: load-balancer
ms.devlang: na
ms.topic: how-to
ms.custom: seodec18
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/04/2021
ms.author: allensu
ms.openlocfilehash: 2d8d5d84d32cdb8cc813d033f3f3fbb453b538fe
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101739922"
---
# <a name="configure-the-distribution-mode-for-azure-load-balancer"></a>Настройка режима распределения для Azure Load Balancer

Azure Load Balancer поддерживает два режима распространения для распределения трафика между приложениями:

* На основе хэша
* Сходство исходного IP-адреса

Из этой статьи вы узнаете, как настроить режим распределения для Azure Load Balancer.


## <a name="configure-distribution-mode"></a>Настройка режима распространения

---

# <a name="azure-portal"></a>[**Портал Azure**](#tab/azure-portal)

Можно изменить конфигурацию режима распределения, изменив правило балансировки нагрузки на портале.

1. Войдите в портал Azure и выберите группу ресурсов, содержащую подсистему балансировки нагрузки, которую вы хотите изменить, щелкнув **группы ресурсов**.
2. На экране "Обзор подсистемы балансировки нагрузки" выберите **правила балансировки нагрузки** в разделе " **Параметры**".
3. На экране правила балансировки нагрузки выберите правило балансировки нагрузки, которое вы хотите изменить в режиме распространения.
4. В правиле режим распределения изменяется путем изменения раскрывающегося списка **сохранение сеанса** . 

Доступны следующие варианты: 

* **Нет (на основе хэша)** . указывает, что последовательные запросы от одного клиента могут обрабатываться любой виртуальной машиной.
* **IP-адрес клиента (два кортежа с сопоставлением ИСХОДНОГО IP** -адреса). указывает, что последовательные запросы с одного и того же IP-адресов клиента будут обрабатываться одной и той же виртуальной машиной.
* **IP-адрес и протокол клиента (сопоставление исходного IP-адреса с тремя кортежами)** . указывает, что последовательные запросы с одинаковой КОМБИНАЦИЕЙ IP-адресов и протоколов клиента будут обрабатываться одной и той же виртуальной машиной.

5. Выберите режим распределения и нажмите кнопку **сохранить**.

:::image type="content" source="./media/load-balancer-distribution-mode/session-persistence.png" alt-text="Измените сохраняемость сеанса в правиле подсистемы балансировки нагрузки." border="true":::


# <a name="powershell"></a>[**PowerShell**](#tab/azure-powershell)

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Используйте PowerShell, чтобы изменить параметры распределения балансировки нагрузки в существующем правиле балансировки нагрузки. Следующая команда обновляет режим распределения: 

```azurepowershell-interactive
$lb = Get-AzLoadBalancer -Name MyLoadBalancer -ResourceGroupName MyResourceGroupLB
$lb.LoadBalancingRules[0].LoadDistribution = 'default'
Set-AzLoadBalancer -LoadBalancer $lb
```

Установите значение `LoadDistribution` элемента для требуемого типа балансировки нагрузки. 

* Укажите **SourceIP** для балансировки нагрузки двух кортежей (исходный IP-адрес и IP-адрес назначения). 

* Укажите **SourceIPProtocol** для балансировки нагрузки трех кортежей (исходный IP-адрес, IP-адрес назначения и тип протокола). 

* Укажите **значение по** умолчанию для режима балансировки нагрузки с пятью кортежами.

# <a name="cli"></a>[**CLI**](#tab/azure-cli)

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

Используйте Azure CLI, чтобы изменить параметры распределения балансировки нагрузки в существующем правиле балансировки нагрузки.  Следующая команда обновляет режим распределения:

```azurecli-interactive
az network lb rule update \
    --lb-name myLoadBalancer \
    --load-distribution Default \
    --name myHTTPRule \
    --resource-group myResourceGroupLB 
```
Задайте значение `--load-distribution` для требуемого типа балансировки нагрузки.

* Укажите **SourceIP** для балансировки нагрузки двух кортежей (исходный IP-адрес и IP-адрес назначения). 

* Укажите **SourceIPProtocol** для балансировки нагрузки трех кортежей (исходный IP-адрес, IP-адрес назначения и тип протокола). 

* Укажите **значение по** умолчанию для режима балансировки нагрузки с пятью кортежами.

Дополнительные сведения о команде, используемой в этой статье, см. в разделе [AZ Network фунтов правило UPDATE](/cli/azure/network/lb/rule#az_network_lb_rule_update) .

---

## <a name="next-steps"></a>Дальнейшие действия

* [Обзор Azure Load Balancer](load-balancer-overview.md)
* [Создание балансировщика нагрузки для Интернета в Resource Manager с помощью PowerShell](quickstart-load-balancer-standard-public-powershell.md)
* [Настройка параметров времени ожидания простоя TCP для подсистемы балансировки нагрузки](load-balancer-tcp-idle-timeout.md)