---
title: Служба Azure Front Door | Документация Майкрософт
description: В статье представлен обзор службы Azure Front Door.
services: frontdoor
author: duongau
ms.service: frontdoor
ms.topic: overview
ms.workload: infrastructure-services
ms.date: 03/09/2021
ms.author: duau
ms.openlocfilehash: 0ba34b298fad21656a63c813057ed0c69db6c7f1
ms.sourcegitcommit: 73fb48074c4c91c3511d5bcdffd6e40854fb46e5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/31/2021
ms.locfileid: "106064891"
---
# <a name="what-is-azure-front-door"></a>Что такое Azure Front Door?

> [!IMPORTANT]
> Эта документация предназначена для Azure Front Door. Ищете информацию об Azure Front Door цен. категории "Стандартный" или "Премиум" (предварительная версия)? см. [здесь](standard-premium/overview.md).

Azure Front Door — это глобальная масштабируемая точка входа, которая использует глобальную промежуточную подсеть Майкрософт для создания быстрых, безопасных и масштабируемых веб-приложений. С помощью службы Front Door вы можете преобразовать глобальные пользовательские и корпоративные приложения в современные, надежные, высокопроизводительные и персонализированные приложения с содержимым, которое охватывает глобальную аудиторию с помощью Azure.

<p align="center">
  <img src="./media/front-door-overview/front-door-visual-diagram.png" alt="Front Door architecture" width="600" title="Azure Front Door">
</p>

Служба Front Door работает на протоколах 7-го уровня (уровня HTTP/HTTPS), используя протокол произвольной передачи с разделенным протоколом TCP, а также глобальную сеть Майкрософт, чтобы улучшить возможность глобального подключения. Основываясь на методе маршрутизации, вы можете убедиться, что служба Front Door будет направлять клиентские запросы в самую быструю и доступную серверную часть приложения. Серверная часть приложения — это любая служба, размещенная в среде Azure или за ее пределами, с доступом в Интернет. Служба Front Door предоставляет целый ряд [методов маршрутизации трафика](front-door-routing-methods.md) и [параметров мониторинга работоспособности сервера](front-door-health-probes.md), подходящих для приложений с различными потребностями и сценариями автоматического перехода на другой ресурс. Так же как и [Диспетчер трафика](../traffic-manager/traffic-manager-overview.md), служба Front Door устойчива к сбоям, в том числе к сбоям всего региона Azure.

>[!NOTE]
> Azure предоставляет набор полностью управляемых решений балансировки нагрузки для пользовательских сценариев. 
> * Если вам необходима глобальная маршрутизация на основе DNS и вам **не** нужны обработка подключений по протоколу TLS (разгрузка SSL), обработка прикладного уровня HTTP- или HTTPS-запросов, вам стоит обратить внимание на [Диспетчер трафика](../traffic-manager/traffic-manager-overview.md). 
> * Если вы хотите распределить нагрузку между серверами в регионе на уровне приложения, ознакомьтесь с возможностями [Шлюза приложений](../application-gateway/overview.md).
> * Чтобы выполнить балансировку нагрузки на сетевом уровне, ознакомьтесь с [Load Balancer](../load-balancer/load-balancer-overview.md). 
> 
> В комплексных сценариях может быть целесообразно объединить эти решения.
> Сравнение параметров балансировки нагрузки Azure см. в статье [Overview of load-balancing options in Azure](/azure/architecture/guide/technology-choices/load-balancing-overview) (Общие сведения о параметрах балансировки нагрузки в Azure).

## <a name="why-use-azure-front-door"></a>Зачем использовать службу Azure Front Door?

С помощью Front Door можно создавать и использовать динамическое веб-приложение и статическое содержимое, а также горизонтально увеличивать их масштаб. Front Door помогает определять, отслеживать и контролировать глобальную маршрутизацию для вашего веб-трафика. При этом оптимизация конечных пользователей верхнего уровня позволяет достичь наилучшей производительности, а быстрая глобальная отработка отказа — высокой надежности.

Служба Front Door включает следующие основные функции.

* Повышение производительности приложения с помощью **[протокола произвольной передачи](front-door-routing-architecture.md#anycast)** на основе **[разделенного протокола TCP](front-door-routing-architecture.md#splittcp)** .

* Интеллектуальный мониторинг **[пробы работоспособности](front-door-health-probes.md)** для серверных ресурсов.

*  Маршрутизация **[на основе URL-путей](front-door-route-matching.md)** для запросов.

* Включает размещение нескольких веб-сайтов для эффективной инфраструктуры приложений. 

* **[Сходство сеансов](front-door-routing-methods.md#affinity)** на основе файлов cookie.

* **[Разгрузка SSL](front-door-custom-domain-https.md)** и управление сертификатами.

* Определение **[личного домена](front-door-custom-domain.md)** . 

* Защита приложений с интегрированным **[Брандмауэром веб-приложений (WAF)](../web-application-firewall/overview.md)** .

* Перенаправление трафика HTTP на HTTPS с помощью **[перенаправления URL-адреса](front-door-url-redirect.md)** .

* Пользовательский путь перенаправления с помощью **[переопределения URL-адреса](front-door-url-rewrite.md)** .

* Встроенная поддержка сквозного подключения по протоколу IPv6 и **[протоколу HTTP/2](front-door-http2.md)** .

## <a name="pricing"></a>Цены

Сведения о ценах на службу Front Door см. на странице [расценок Azure Front Door Service](https://azure.microsoft.com/pricing/details/frontdoor/). См. [Соглашение об уровне обслуживания для Azure Front Door](https://azure.microsoft.com/en-us/support/legal/sla/frontdoor/v1_0/).

## <a name="whats-new"></a>Новые возможности

Подпишитесь на RSS-канал и просматривайте последние обновления компонентов для Azure Front Door на странице [Обновления Azure](https://azure.microsoft.com/updates/?category=networking&query=Azure%20Front%20Door).

## <a name="next-steps"></a>Дальнейшие действия

- Дополнительные сведения о [создании Front Door](quickstart-create-front-door.md).
- Дополнительные сведения о том, [как работает Front Door](front-door-routing-architecture.md).
