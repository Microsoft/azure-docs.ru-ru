---
title: Что такое Azure Load Balancer?
titleSuffix: Azure Load Balancer
description: Обзор функций Azure Load Balancer, его архитектуры и реализации. Узнайте, как работает Load Balancer и его использование в облаке.
services: load-balancer
documentationcenter: na
author: asudbring
ms.service: load-balancer
ms.devlang: na
ms.topic: overview
ms.custom: seodec18
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 1/25/2021
ms.author: allensu
ms.openlocfilehash: f847ef4f4490b64212e4cdcce3f9a89e94c7cd37
ms.sourcegitcommit: 73fb48074c4c91c3511d5bcdffd6e40854fb46e5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/31/2021
ms.locfileid: "106058601"
---
# <a name="what-is-azure-load-balancer"></a>Что такое Azure Load Balancer?

*Балансировка нагрузки* обозначает процесс равномерного распределения нагрузки (входящего сетевого трафика) в группе серверных ресурсов или серверов. 

Служба Azure Load Balancer работает на уровне 4 модели OSI (Open Systems Interconnection). Она служит единственной точкой взаимодействия с клиентами. Load Balancer распределяет входящие потоки, поступающие на внешний интерфейс подсистемы балансировки нагрузки, в экземпляры серверных пулов. Эти потоки соответствуют настроенным правилам балансировки нагрузки и пробам работоспособности. В серверный пул могут входить Виртуальные машины Azure или экземпляры масштабируемого набора виртуальных машин.

**[Общедоступный Load Balancer](./components.md#frontend-ip-configurations)** обеспечивает исходящие подключения для виртуальных машин в виртуальной сети. Эти подключения выполняются путем преобразования их частных IP-адресов в общедоступные. Общедоступные Load Balancer используются для балансировки трафика, направленного из Интернета к виртуальным машинам.

**[Внутренний (или частный) Load Balancer](./components.md#frontend-ip-configurations)** используется там, где частные IP-адреса необходимы только в интерфейсной части. Внутренние Load Balancer используются для балансировки трафика внутри виртуальной сети. В гибридном сценарии ко внешнему интерфейсу Load Balancer можно подключиться из локальной сети.

<p align="center">
  <img src="./media/load-balancer-overview/load-balancer.svg" alt="Figure depicts both public and internal load balancers directing traffic to port 80 on multiple servers on a Web tier and port 443 on multiple servers on a business tier." width="512" title="Azure Load Balancer">
</p>

*Рисунок. Балансировка многоуровневых приложений с помощью общедоступных и внутренних Load Balancer*

Дополнительные сведения об отдельных компонентах Load Balancer см. в статье [Компоненты Azure Load Balancer](./components.md).

>[!NOTE]
> Azure предоставляет набор полностью управляемых решений балансировки нагрузки для пользовательских сценариев. 
> * Если вам необходима глобальная маршрутизация на основе DNS и вам **не** нужны обработка подключений по протоколу TLS (разгрузка SSL), обработка прикладного уровня HTTP- или HTTPS-запросов, вам стоит обратить внимание на [Диспетчер трафика](../traffic-manager/traffic-manager-overview.md). 
> * Если вы хотите распределить нагрузку между серверами в регионе на уровне приложения, ознакомьтесь с возможностями [Шлюза приложений](../application-gateway/overview.md).
> * Если нужно оптимизировать глобальную маршрутизацию для вашего веб-трафика, а также обеспечить лучшую производительность и надежность для пользователей посредством быстрой глобальной отработки отказа, ознакомьтесь с возможностями службы [Front Door](../frontdoor/front-door-overview.md).
> 
> В комплексных сценариях может быть целесообразно объединить эти решения.
> Сравнение параметров балансировки нагрузки Azure см. в статье [Overview of load-balancing options in Azure](/azure/architecture/guide/technology-choices/load-balancing-overview) (Общие сведения о параметрах балансировки нагрузки в Azure).


## <a name="why-use-azure-load-balancer"></a>Зачем нужен Azure Load Balancer?
C помощью Azure Load Balancer можно масштабировать приложения и создавать службы с высоким уровнем доступности. Load Balancer поддерживает сценарии как входящих так и исходящих подключений. Load Balancer обеспечивает низкую задержку и высокую пропускную способность, а также увеличение масштаба до миллионов потоков для всех приложений, которые используют протоколы TCP и UDP.

Основные сценарии, которые можно выполнить с помощью подсистемы Azure Load Balancer (цен. категория "Стандартный"):

- Балансировка нагрузки **[внутреннего](./quickstart-load-balancer-standard-internal-portal.md)** и **[внешнего](./quickstart-load-balancer-standard-public-portal.md)** трафика на виртуальных машинах Azure.

- Повышение доступности за счет распределения ресурсов **[в пределах зон](./tutorial-load-balancer-standard-public-zonal-portal.md)** и **[между](./tutorial-load-balancer-standard-public-zone-redundant-portal.md)** ними.

- Настройка **[исходящего подключения](./load-balancer-outbound-connections.md)** для виртуальных машин Azure.

- Использование **[проб работоспособности](./load-balancer-custom-probe-overview.md)** для мониторинга ресурсов с балансировкой нагрузки.

- Использование **[перенаправления портов](./tutorial-load-balancer-port-forwarding-portal.md)** , чтобы получить доступ к виртуальным машинам в виртуальной сети по общедоступному IP-адресу и порту.

- Включение поддержки **[балансировки нагрузки](../virtual-network/virtual-network-ipv4-ipv6-dual-stack-standard-load-balancer-powershell.md)** **[IPv6](../virtual-network/ipv6-overview.md)** .

- Load Balancer уровня "Стандартный" предоставляет многомерные метрики через [Azure Monitor](../azure-monitor/overview.md).  Эти метрики можно отфильтровать, сгруппировать и разбить по предоставленным измерениям.  Они предоставляют текущие и исторические сведения о производительности и работоспособности службы. [Аналитика для Azure Load Balancer](./load-balancer-insights.md) предоставляет предварительно настроенную панель мониторинга с полезными визуализациями для этих метрик.  Также поддерживается служба "Работоспособность ресурсов". Дополнительные сведения см. в статье **[Диагностика Load Balancer (цен. категория "Стандартный") с помощью метрик, оповещений и сведений о работоспособности ресурсов](load-balancer-standard-diagnostics.md)** .

- Службы балансировки нагрузки на **[нескольких портах, нескольких IP-адресах или обоих](./load-balancer-multivip-overview.md)** .

- Перемещение **[внутренних](./move-across-regions-internal-load-balancer-portal.md)** и **[внешних](./move-across-regions-external-load-balancer-portal.md)** ресурсов Load Balancer в регионах Azure.

- Балансировка нагрузки потоков TCP и UDP на всех портах одновременно с использованием **[Портов высокой доступности](./load-balancer-ha-ports-overview.md)** .

### <a name="secure-by-default"></a><a name="securebydefault"></a>Обеспечение безопасности по умолчанию

* Служба Load Balancer (цен. категория "Стандартный") построена на основе модели безопасности сети "Никому не доверяй".

* Load Balancer (цен. категория "Стандартный") защищен по умолчанию и является частью вашей виртуальной сети. Это виртуальная сеть является частной и изолированной сетью.  

* Load Balancer (цен. категории "Стандартный") и стандартные общедоступные IP-адреса закрыты для входящих подключений, если их не открыть с помощью групп безопасности сети. Группы безопасности сети используются для явного разрешения допустимого трафика.  Если у вас нет NSG в подсети или сетевого интерфейса ресурса виртуальной машины, трафик не может достичь этого ресурса. Дополнительные сведения о группах безопасности сети и способах их применения в вашем сценарии см. в статье [Группы безопасности сети](../virtual-network/network-security-groups-overview.md).

* В Load Balancer (цен. категория "Базовый") по умолчанию открыт доступ к Интернету. 

* Load Balancer не хранит данные клиентов.

## <a name="pricing-and-sla"></a>Цены и соглашение об уровне обслуживания

Сведения о ценах на Load Balancer (цен. категория "Стандартный") см. на [странице цен Load Balancer](https://azure.microsoft.com/pricing/details/load-balancer/).
Load Balancer (цен. категория "Базовый") предоставляется бесплатно.
См. [соглашение об уровне обслуживания для Load Balancer](https://aka.ms/lbsla). Load Balancer (цен. категория "Базовый") не имеет соглашения об уровне обслуживания (SLA).

## <a name="whats-new"></a>Новые возможности

Подпишитесь на RSS-канал и просматривайте последние обновления компонентов для Azure Load Balancer на странице [Обновления Azure](https://azure.microsoft.com/updates/?category=networking&query=load%20balancer).

## <a name="next-steps"></a>Дальнейшие действия

Сведения о том, как приступить к работе с Load Balancer, см. в статье [Краткое руководство. Создание общедоступной подсистемы балансировки нагрузки с помощью портала Azure для распределения нагрузки между виртуальными машинами](quickstart-load-balancer-standard-public-portal.md).

Дополнительные сведения об ограничениях и компонентах Azure Load Balancer см. в статьях [Компоненты Azure Load Balancer](./components.md) и [Концепции Azure Load Balancer](./concepts.md).