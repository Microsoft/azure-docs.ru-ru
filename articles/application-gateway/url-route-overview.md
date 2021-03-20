---
title: Общие сведения о маршрутизации содержимого на основе URL-адресов с помощью службы "Шлюз приложений Azure"
description: В этой статье представлен обзор маршрутизации содержимого на основе URL-адресов шлюза приложений Azure, конфигурации UrlPathMap и правила PathBasedRouting.
services: application-gateway
author: vhorne
ms.service: application-gateway
ms.date: 09/10/2019
ms.author: victorh
ms.topic: conceptual
ms.openlocfilehash: a9b2e8148586ec58ea6a7a033099e726920857b6
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "84987932"
---
# <a name="url-path-based-routing-overview"></a>Общие сведения о маршрутизации на основе URL-путей

Маршрутизация на основе URL-путей позволяет направлять трафик в пулы тыловых серверов в зависимости от URL-путей поступающих запросов. 

Один из сценариев — это маршрутизация запросов содержимого различных типов в различные пулы тыловых серверов.

В следующем примере шлюз приложений обслуживает трафик веб-сайта contoso.com с трех пулов тыловых серверов, например VideoServerPool, ImageServerPool и DefaultServerPool.

![imageURLroute](./media/application-gateway-url-route-overview/figure1.png)

Запросы для http\://contoso.com/video/* направляются в VideoServerPool, а запросы для http\://contoso.com/images/* — в ImageServerPool. Если ни один из шаблонов пути не подходит, выбирается пул DefaultServerPool.

> [!IMPORTANT]
> Для номера SKU v1 правила обрабатываются в том порядке, в котором они указаны на портале. Если базовый прослушиватель стоит первым в списке и совпадает с входящим запросом, он будет обрабатываться прослушивателем. Для SKU версии 2 точные совпадения имеют более высокий приоритет. Однако настоятельно рекомендуется настроить многосайтовые прослушиватели, прежде чем настраивать базовый прослушиватель. Это гарантирует, что трафик будет перенаправляться на правильный внутренний сервер.

## <a name="urlpathmap-configuration-element"></a>Элемент конфигурации UrlPathMap

Элемент urlPathMap используется для определения шаблонов пути при сопоставлении с пулом тыловых серверов. Ниже приведен пример кода, который является фрагментом элемента urlPathMap из файла шаблона.

```json
"urlPathMaps": [{
    "name": "{urlpathMapName}",
    "id": "/subscriptions/{subscriptionId}/../microsoft.network/applicationGateways/{gatewayName}/urlPathMaps/{urlpathMapName}",
    "properties": {
        "defaultBackendAddressPool": {
            "id": "/subscriptions/    {subscriptionId}/../microsoft.network/applicationGateways/{gatewayName}/backendAddressPools/{poolName1}"
        },
        "defaultBackendHttpSettings": {
            "id": "/subscriptions/{subscriptionId}/../microsoft.network/applicationGateways/{gatewayName}/backendHttpSettingsList/{settingname1}"
        },
        "pathRules": [{
            "name": "{pathRuleName}",
            "properties": {
                "paths": [
                    "{pathPattern}"
                ],
                "backendAddressPool": {
                    "id": "/subscriptions/{subscriptionId}/../microsoft.network/applicationGateways/{gatewayName}/backendAddressPools/{poolName2}"
                },
                "backendHttpsettings": {
                    "id": "/subscriptions/{subscriptionId}/../microsoft.network/applicationGateways/{gatewayName}/backendHttpsettingsList/{settingName2}"
                }
            }
        }]
    }
}]
```

### <a name="pathpattern"></a>PathPattern

PathPattern — это список шаблонов пути для сопоставления. Каждый шаблон должен начинаться с косой черты (/). Символ * может быть только в конце после косой черты. Строка, передаваемая в сопоставитель путей, не должна содержать текст после первого символа "?" или "#", и сами эти символы не допускаются. В остальных случаях все допустимые символы в URL-адресе допускаются и в PathPattern.

Поддерживаемые шаблоны зависят от того, какой Шлюз приложений развернут: версий 1 или 2.

#### <a name="v1"></a>Версия 1

В правилах для путей не учитывается регистр.

|Шаблон пути версии 1  |Поддерживается?  |
|---------|---------|
|`/images/*`     |да|
|`/images*`     |да|
|`/images/*.jpg`     |Нет|
|`/*.jpg`     |Нет|
|`/Repos/*/Comments/*`     |Нет|
|`/CurrentUser/Comments/*`     |да|

#### <a name="v2"></a>Версия 2

В правилах для путей не учитывается регистр.

|Шаблон пути версии 2  |Поддерживается?  |
|---------|---------|
|`/images/*`     |да|
|`/images*`     |да|
|`/images/*.jpg`     |Нет|
|`/*.jpg`     |Нет|
|`/Repos/*/Comments/*`     |Нет|
|`/CurrentUser/Comments/*`     |да|

Дополнительные сведения см. в статье [Resource Manager template using URL-based routing](https://azure.microsoft.com/documentation/templates/201-application-gateway-url-path-based-routing) (Шаблон Resource Manager с использованием маршрутизации на основе URL-адресов).

## <a name="pathbasedrouting-rule"></a>Правило PathBasedRouting

Правило RequestRoutingRule типа PathBasedRouting используется для привязки прослушивателя к urlPathMap. Все получаемые запросы для этого прослушивателя распределяются согласно политике, указанной в urlPathMap.
Фрагмент правила PathBasedRouting:

```json
"requestRoutingRules": [
    {

"name": "{ruleName}",
"id": "/subscriptions/{subscriptionId}/../microsoft.network/applicationGateways/{gatewayName}/requestRoutingRules/{ruleName}",
"properties": {
    "ruleType": "PathBasedRouting",
    "httpListener": {
        "id": "/subscriptions/{subscriptionId}/../microsoft.network/applicationGateways/{gatewayName}/httpListeners/<listenerName>"
    },
    "urlPathMap": {
        "id": "/subscriptions/{subscriptionId}/../microsoft.network/applicationGateways/{gatewayName}/urlPathMaps/{urlpathMapName}"
    }

}
    }
]
```

## <a name="next-steps"></a>Дальнейшие действия

Ознакомившись с маршрутизацией на основе URL-адресов, создайте шлюз приложений с соответствующими правилами маршрутизации, как указано в статье о [создании шлюза приложений с помощью маршрутизации на основе URL-адресов](create-url-route-portal.md) .
