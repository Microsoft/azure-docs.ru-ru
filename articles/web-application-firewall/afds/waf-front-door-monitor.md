---
title: Мониторинг и ведение журнала брандмауэра веб-приложения Azure
description: Знакомство с брандмауэром веб-приложения (WAF) с мониторингом и ведением журнала FrontDoor
author: vhorne
ms.service: web-application-firewall
ms.topic: article
services: web-application-firewall
ms.date: 06/09/2020
ms.author: victorh
ms.openlocfilehash: 596374d4f3f188e08a10bd25b36b178cc79a6e57
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "84808957"
---
# <a name="azure-web-application-firewall-monitoring-and-logging"></a>Мониторинг и ведение журнала брандмауэра веб-приложения Azure

Мониторинг и ведение журнала брандмауэра веб-приложения Azure (WAF) осуществляется с помощью ведения журналов и интеграции с журналами Azure Monitor и Azure Monitor.

## <a name="azure-monitor"></a>Azure Monitor

WAF с журналом FrontDoor интегрируется с [Azure Monitor](../../azure-monitor/overview.md). Azure Monitor позволяет вести мониторинг диагностических данных, включая оповещения и журналы WAF. Вы можете настроить мониторинг WAF в ресурсе передней дверцы на портале на вкладке **Диагностика** или непосредственно в службе Azure Monitor.

В портал Azure выберите тип ресурса "Передняя дверь". На  / вкладке **метрики** мониторинга слева можно добавить **вебаппликатионфиреваллрекуесткаунт** для отслеживания количества запросов, соответствующих правилам WAF. Настраиваемые фильтры можно создавать на основе типов действий и имен правил.

:::image type="content" source="../media/waf-frontdoor-monitor/waf-frontdoor-metrics.png" alt-text="вафметрикс ":::

## <a name="logs-and-diagnostics"></a>Журналы и диагностика

WAF с передней дверцей предоставляет подробные отчеты по каждой обнаруженной угрозе. Функция ведения журналов интегрирована с журналами системы диагностики Azure, и оповещения записываются в формате JSON. Эти журналы можно интегрировать с [журналами Azure Monitor](../../azure-monitor/insights/azure-networking-analytics.md).

![вафдиаг](../media/waf-frontdoor-monitor/waf-frontdoor-diagnostics.png)

[Фронтдуракцесслог](../../frontdoor/front-door-diagnostics.md) регистрирует все запросы. Фронтдурвебаппликатионфиревалллог регистрирует любой запрос, соответствующий правилу WAF, имеющему следующую схему:

| Свойство  | Описание |
| ------------- | ------------- |
|Действие|Действие, предпринимаемое для запроса|
| ClientIp | IP-адрес отправившего запрос клиента. Если в запросе был указан заголовок X-Forwardd-for, то IP-адрес клиента выбирается из поля заголовка. |
| клиентпорт | IP-порт клиента, который сделал запрос. |
| Сведения|Дополнительные сведения о сопоставленном запросе |
|| Матчвариабленаме: имя параметра HTTP, соответствующее запросу, например имена заголовков|
|| Матчвариаблевалуе: значения, вызвавшие сопоставление|
| Узел | Заголовок узла сопоставленного запроса |
| Политика | Имя политики WAF, которой соответствует запрос. |
| полицимоде | Режим работы политики WAF. Возможные значения — "предотвращение" и "Обнаружение" |
| RequestUri | Полный URI сопоставленного запроса. |
| RuleName | Имя правила WAF, которому соответствует запрос. |
| соккетип | Исходный IP-адрес, видимый WAF. Этот IP-адрес основан на сеансе TCP, независимо от любых заголовков запроса.|
| TrackingReference | Это уникальная эталонная строка, которая идентифицирует обслуженный Front Door запрос. Она же отправляется клиенту в заголовке X-Azure-Ref. Эта строка нужна для поиска в журналах доступа сведений о конкретном запросе. |

Следующий пример запроса возвращает журналы WAF для заблокированных запросов:

``` WAFlogQuery
AzureDiagnostics
| where ResourceType == "FRONTDOORS" and Category == "FrontdoorWebApplicationFirewallLog"
| where action_s == "Block"

```

Ниже приведен пример зарегистрированного запроса в журнале WAF:

``` WAFlogQuerySample
{
    "time":  "2020-06-09T22:32:17.8376810Z",
    "category": "FrontdoorWebApplicationFirewallLog",
    "operationName": "Microsoft.Network/FrontDoorWebApplicationFirewallLog/Write",
    "properties":
    {
        "clientIP":"xxx.xxx.xxx.xxx",
        "clientPort":"52097",
        "socketIP":"xxx.xxx.xxx.xxx",
        "requestUri":"https://wafdemofrontdoorwebapp.azurefd.net:443/?q=%27%20or%201=1",
        "ruleName":"Microsoft_DefaultRuleSet-1.1-SQLI-942100",
        "policy":"WafDemoCustomPolicy",
        "action":"Block",
        "host":"wafdemofrontdoorwebapp.azurefd.net",
        "trackingReference":"08Q3gXgAAAAAe0s71BET/QYwmqtpHO7uAU0pDRURHRTA1MDgANjMxNTAwZDAtOTRiNS00YzIwLTljY2YtNjFhNzMyOWQyYTgy",
        "policyMode":"prevention",
        "details":
            {
            "matches":
                [{
                "matchVariableName":"QueryParamValue:q",
                "matchVariableValue":"' or 1=1"
                }]
            }
     }
}
```

Следующий пример запроса возвращает записи Акцесслогс:

``` AccessLogQuery
AzureDiagnostics
| where ResourceType == "FRONTDOORS" and Category == "FrontdoorAccessLog"

```

Ниже приведен пример зарегистрированного запроса в журнале доступа:

``` AccessLogSample
{
"time": "2020-06-09T22:32:17.8383427Z",
"category": "FrontdoorAccessLog",
"operationName": "Microsoft.Network/FrontDoor/AccessLog/Write",
 "properties":
    {
    "trackingReference":"08Q3gXgAAAAAe0s71BET/QYwmqtpHO7uAU0pDRURHRTA1MDgANjMxNTAwZDAtOTRiNS00YzIwLTljY2YtNjFhNzMyOWQyYTgy",
    "httpMethod":"GET",
    "httpVersion":"2.0",
    "requestUri":"https://wafdemofrontdoorwebapp.azurefd.net:443/?q=%27%20or%201=1",
    "requestBytes":"715",
    "responseBytes":"380",
    "userAgent":"Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/85.0.4157.0 Safari/537.36 Edg/85.0.531.1",
    "clientIp":"xxx.xxx.xxx.xxx",
    "socketIp":"xxx.xxx.xxx.xxx",
    "clientPort":"52097",
    "timeTaken":"0.003",
    "securityProtocol":"TLS 1.2",
    "routingRuleName":"WAFdemoWebAppRouting",
    "rulesEngineMatchNames":[],
    "backendHostname":"wafdemowebappuscentral.azurewebsites.net:443",
    "sentToOriginShield":false,
    "httpStatusCode":"403",
    "httpStatusDetails":"403",
    "pop":"SJC",
    "cacheStatus":"CONFIG_NOCACHE"
    }
}

```

## <a name="next-steps"></a>Дальнейшие действия

- Дополнительные сведения о [передней дверце](../../frontdoor/front-door-overview.md).
