---
title: Номера SKU для Azure Load Balancer
description: Общие сведения о номерах SKU для Azure Load Balancer
services: load-balancer
documentationcenter: na
author: asudbring
ms.service: load-balancer
ms.devlang: na
ms.topic: reference
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 05/01/2020
ms.author: allensu
ms.openlocfilehash: 874ecfc8c1c50816916fb0b04975477a1cbe0a71
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "94698093"
---
# <a name="azure-load-balancer-skus"></a>Номера SKU для Azure Load Balancer

Azure Load Balancer предоставляется с двумя номерами SKU.

## <a name="sku-comparison"></a><a name="skus"></a> Сравнение SKU

Load Balancer поддерживает SKU уровня "Базовый" и "Стандартный". Эти SKU различаются по масштабу сценария, возможностям и цене. Любой сценарий, который можно выполнить в рамках Load Balancer уровня "Базовый", также можно создать с помощью Load Balancer уровня "Стандартный".

Ознакомьтесь со следующей таблицей, чтобы сравнить сведения и понять различия. Дополнительные сведения см. в статье [Обзор Azure Load Balancer уровня "Стандартный"](./load-balancer-overview.md) на портале Azure.

>[!NOTE]
> Корпорация Майкрософт рекомендует использовать Load Balancer уровня "Стандартный".
Изолированные виртуальные машины, наборы доступности и масштабируемые наборы виртуальных машин можно подключить только к одному SKU, но никогда к обоим. При использовании общедоступных IP-адресов номера SKU Load Balancer и IP-адреса должны совпадать. Номера SKU Load Balancer и общедоступные IP-адреса неизменяемы.

| | Load Balancer (цен. категория "Стандартный") | Azure Load Balancer, категория "Базовый" |
| --- | --- | --- |
| **[Размер серверного пула](../azure-resource-manager/management/azure-subscription-service-limits.md#load-balancer)** | Поддерживается до 1000 экземпляров. | Поддерживается до 300 экземпляров. |
| **Конечные точки серверного пула** | Виртуальные машины или масштабируемые наборы виртуальных машин в одной виртуальной сети. | Виртуальные машины в одной группе доступности или масштабируемом наборе виртуальных машин. |
| **[Зонды работоспособности](./load-balancer-custom-probe-overview.md#types)** | TCP, HTTP, HTTPS | TCP, HTTP |
| **[Реакция на сбой зонда работоспособности](./load-balancer-custom-probe-overview.md#probedown)** | TCP-подключения остаются активными при сбое проверки экземпляра __и__ сбое всех проверок. | TCP-подключения остаются активными при сбое проверки экземпляра. Все TCP-подключения завершаются, когда все пробы отключены. |
| **Зоны доступности** | Избыточные между зонами и зональные внешние интерфейсы для входящего и исходящего трафика. | Недоступно |
| **Диагностика** | [Многомерные метрики Azure Monitor](./load-balancer-standard-diagnostics.md) | [Журналы Azure Monitor](./load-balancer-monitor-log.md) |
| **Порты высокого уровня доступности** | [Доступно для внутреннего Load Balancer](./load-balancer-ha-ports-overview.md) | Недоступно |
| **Обеспечение безопасности по умолчанию** | Закрыто для входящих потоков, если иное не разрешено группой безопасности сети. Внутренний трафик из виртуальной сети во внутреннюю подсистему балансировки нагрузки разрешен. | Открыты по умолчанию. Группа безопасности сети необязательна. |
| **Правила для исходящего трафика** | [Декларативная конфигурация NAT для исходящего трафика](./load-balancer-outbound-connections.md#outboundrules) | Недоступно |
| **Сброс подключений TCP в режиме простоя** | [Доступно для любого правила](./load-balancer-tcp-reset.md) | Недоступно |
| **[Несколько внешних интерфейсов](./load-balancer-multivip-overview.md)** | Входящий и [исходящий](./load-balancer-outbound-connections.md). | Только входящий |
| **Операции управления** | Большинство операций длятся менее 30 секунд | Обычно 60–90 и более секунд |
| **Соглашение об уровне обслуживания** | [99,99 %](https://azure.microsoft.com/support/legal/sla/load-balancer/v1_0/) | Недоступно | 

Дополнительные сведения см. в статье [Ограничения подсистемы балансировки нагрузки Azure](../azure-resource-manager/management/azure-subscription-service-limits.md#load-balancer). Для Load Balancer уровня "Стандартный" см. [общие сведения](./load-balancer-overview.md), а также [сведения о ценах](https://aka.ms/lbpricing) и [Соглашении об уровне обслуживания](https://aka.ms/lbsla).

## <a name="limitations"></a>Ограничения

- Ценовую категорию изменить нельзя. Вы не можете изменить ценовую категорию для существующего ресурса.
- Изолированный ресурс виртуальной машины, ресурс группы доступности или ресурс масштабируемого набора виртуальных машин могут ссылаться на один SKU, но никогда на оба.
- [Операции перемещения.](../azure-resource-manager/management/move-resource-group-and-subscription.md)
  - Операции перемещения группы ресурсов (в рамках одной подписки) **поддерживаются** для Load Balancer ценовой категории "Стандартный" и общедоступных IP-адресов ценовой категории "Стандартный". 
  - [Операции перемещения группы подписок](../azure-resource-manager/management/move-support-resources.md) **не** поддерживаются для ресурсов Load Balancer ценовой категории "Стандартный" и общедоступных IP-адресов ценовой категории "Стандартный".

## <a name="next-steps"></a>Дальнейшие действия

- Чтобы приступить к работе с Load Balancer, см. [Create a public Standard Load Balancer](quickstart-load-balancer-standard-public-portal.md) (Создание общедоступного Load Balancer (цен. категория "Стандартный")).
- Дополнительные сведения о [Load Balancer (цен. категория "Стандартный") и зонах доступности](load-balancer-standard-availability-zones.md).
- Дополнительные сведения о [пробах работоспособности](load-balancer-custom-probe-overview.md).
- Дополнительные сведения о [Load Balancer для исходящих подключений](load-balancer-outbound-connections.md).
- Дополнительные сведения об [использовании Load Balancer (цен. категория "Стандартный") с правилами балансировки нагрузки портов высокого уровня доступности](load-balancer-ha-ports-overview.md).
- Узнайте больше о [группах безопасности сети](../virtual-network/network-security-groups-overview.md).