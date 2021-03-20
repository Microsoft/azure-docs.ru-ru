---
title: Общие сведения о защите от ботов с помощью брандмауэра веб-приложения в Шлюзе приложений Azure
titleSuffix: Azure Web Application Firewall
description: В этой статье представлены обзорные сведения о защите от ботов с помощью брандмауэра веб-приложения (WAF) в Шлюзе приложений.
services: web-application-firewall
author: winthrop28
ms.service: web-application-firewall
ms.date: 05/20/2020
ms.author: victorh
ms.topic: conceptual
ms.openlocfilehash: e01f9ac8966223e11ad218af7bf6fbb2462f28f6
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "83714905"
---
# <a name="azure-web-application-firewall-on-azure-application-gateway-bot-protection-overview"></a>Общие сведения о защите от ботов с помощью брандмауэра веб-приложения Azure в Шлюзе приложений Azure

Примерно 20% всего трафика в Интернете создают плохие боты. Они выполняют извлечение данных, сканирование и поиск уязвимостей в веб-приложениях. Если останавливать таких ботов с помощью брандмауэра веб-приложения (WAF), они не смогут наносить вред. Это также устранит создаваемую ими нагрузку на ваши ресурсы и службы, например внутренние серверы и базовую инфраструктуру.

Вы можете включить управляемый набор правил защиты от ботов, чтобы WAF блокировал и (или) сохранял в журнал запросы от известных вредоносных IP-адресов. Эти IP-адреса передаются из канала Microsoft Threat Intelligence. Intelligent Security Graph поддерживает Microsoft Threat Intelligence и используется во множестве служб, включая Центр безопасности Azure.

> [!IMPORTANT]
> Сейчас набор правил для защиты от ботов предоставляется в режиме общедоступной предварительной версии и поддерживается соглашением об уровне обслуживания для предварительной версии. Некоторые функции могут не поддерживаться или их возможности могут быть ограничены. Подробные сведения см. на странице  [Дополнительные условия использования Предварительных версий Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) .

## <a name="use-with-owasp-rulesets"></a>Применение с наборами правил OWASP

Вы можете сочетать правила для защиты от ботов с любым набором правил OWASP (2.2.9, 3.0 или 3.1). Одновременно может использоваться только один набор правил OWASP. Набор правил для защиты от ботов содержит дополнительное правило, которое размещено в отдельном наборе правил. Это правило **Microsoft_BotManagerRuleSet_0.1**, которое вы можете включать или отключать обычным образом, как и другие правила OWASP.

![Набор правил для защиты от ботов](../media/bot-protection-overview/bot-ruleset.png)

## <a name="ruleset-update"></a>Обновление набора правил

Список известных вредоносных IP-адресов в наборе правил для защиты от ботов обновляется несколько раз в день через веб-канал аналитики угроз корпорации Майкрософт, чтобы своевременно реагировать на действия ботов. Веб-приложения будут постоянно защищены, даже когда боты меняют векторы атаки.

## <a name="log-example"></a>Пример записей в журнале

Ниже приведен пример записи в журнале, созданной правилом защиты от ботов:

```
{
        "timeStamp": "0000-00-00T00:00:00+00:00",
            "resourceId": "appgw",
            "operationName": "ApplicationGatewayFirewall",
            "category": "ApplicationGatewayFirewallLog",
            "properties": {
            "instanceId": "vm1",
                "clientIp": "1.2.3.4",
                "requestUri": "/hello.php?arg1=aaaaaaabccc",
                "ruleSetType": "MicrosoftBotProtection",
                "message": "IPReputationTriggered",
                "action": "Blocked",
                "hostname": "example.com",
                "transactionId": "abc",
                "policyId": "waf policy 1",
                "policyScope": "Global",
                "policyScopeName": "Default Policy",
                "engine": "Azwaf"
        }
    }
```

## <a name="next-steps"></a>Дальнейшие действия

- [Настройка защиты от ботов для брандмауэра веб-приложения Azure в Шлюзе приложений Azure (предварительная версия)](bot-protection.md)
