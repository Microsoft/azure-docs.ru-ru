---
title: Часто задаваемые вопросы о брандмауэре веб-приложения Azure в Шлюзе приложений
description: В этой статье содержатся ответы на часто задаваемые вопросы о брандмауэре веб-приложения в Шлюзе приложений
services: web-application-firewall
author: vhorne
ms.service: web-application-firewall
ms.topic: article
ms.date: 05/05/2020
ms.author: victorh
ms.openlocfilehash: 890688dba70a7fa654e97652b3e474b919f9a077
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104585389"
---
# <a name="frequently-asked-questions-for-azure-web-application-firewall-on-application-gateway"></a>Часто задаваемые вопросы о брандмауэре веб-приложения Azure в Шлюзе приложений

В этой статье содержатся ответы на часто задаваемые вопросы о брандмауэре веб-приложения Azure (WAF) и о функциях и функциональности Шлюза приложений. 

## <a name="what-is-azure-waf"></a>Что такое Azure WAF?

Azure WAF — это брандмауэр веб-приложения, который помогает защитить веб-приложения от распространенных угроз, таких как внедрение кода SQL, межсайтовые сценарии и другие веб-атаки. Вы можете определить политику WAF, состоящую из сочетания настраиваемых и управляемых правил для управления доступом к веб-приложениям.

Политику Брандмауэра веб-приложения Azure можно применить к веб-приложениям, расположенным на Шлюзе приложений или Azure Front Doors.

## <a name="what-features-does-the-waf-sku-support"></a>Какие функции поддерживает SKU WAF?

SKU WAF поддерживает все функции, доступные в SKU ценовой категории "Стандартный".

## <a name="how-do-i-monitor-waf"></a>Как выполнять мониторинг WAF?

Мониторинг WAF с помощью ведения журнала диагностики. Дополнительную информацию см. в статье [Ведение журнала диагностики и метрики для шлюза приложений](../../application-gateway/application-gateway-diagnostics.md).

## <a name="does-detection-mode-block-traffic"></a>Блокируется ли трафик в режиме обнаружения?

Нет. Режим обнаружения только регистрирует трафик, активирующий правило WAF.

## <a name="can-i-customize-waf-rules"></a>Могу ли я настроить правила WAF?

Да. Дополнительные сведения см. в разделе [Настройка группы правил и правил WAF](application-gateway-customize-waf-rules-portal.md).

## <a name="what-rules-are-currently-available-for-waf"></a>Какие правила на сегодняшний день доступны в WAF?

В настоящее время WAF поддерживает CRS [2.2.9](application-gateway-crs-rulegroups-rules.md#owasp229), [3.0](application-gateway-crs-rulegroups-rules.md#owasp30)и [3.1](application-gateway-crs-rulegroups-rules.md#owasp31). Эти правила обеспечивают базовый уровень защиты от первых 10 уязвимостей, определенных в открытом проекте безопасности веб-приложений (OWASP): 

* Защита от внедрения кода SQL.
* Защита от межсайтовых сценариев
* Защита от распространенных сетевых атак, в том числе от внедрения команд, несанкционированных HTTP-запросов, разделения HTTP-запросов и включения удаленного файла
* Защита от нарушений протокола HTTP.
* Защита от аномалий протокола HTTP, например отсутствия агента пользователя узла и заголовков accept.
* Защита от программ-роботов, программ-обходчиков и сканеров.
* Обнаружение распространенных неправильных конфигураций приложений (Apache, IIS и т. д.).

Дополнительные сведения см. в статье [10 главных уязвимостей по версии OWASP](https://owasp.org/www-project-top-ten/).

## <a name="what-content-types-does-waf-support"></a>Какие типы содержимого поддерживает WAF?

WAF шлюза приложений поддерживают следующие типы содержимого для управляемых правил:

* приложение/json
* application/xml
* application/x-www-form-urlencoded
* multipart/form-data

И для настраиваемых правил:

* application/x-www-form-urlencoded
* multipart/form-data

## <a name="does-waf-support-ddos-protection"></a>Поддерживает ли WAF защиту от атак DDoS?

Да. Вы можете включить защиту от атак DDos в виртуальной сети, где развертывается шлюз приложений. Эта настройка гарантирует, что служба Защиты от атак DDoS также защищает виртуальный IP-адрес шлюза приложений.

### <a name="does-waf-store-customer-data"></a>WAF ли хранение данных клиента?

Нет, WAF не хранит данные клиента.

## <a name="next-steps"></a>Дальнейшие действия

- Узнайте больше о [брандмауэре веб-приложения Azure](../overview.md).
- Узнайте больше о [Azure Front Door](../../frontdoor/front-door-overview.md).
