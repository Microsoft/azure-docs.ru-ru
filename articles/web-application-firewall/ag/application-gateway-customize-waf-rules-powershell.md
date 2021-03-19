---
title: Настройка правил с помощью PowerShell
titleSuffix: Azure Web Application Firewall
description: В этой статье содержатся сведения о том, как настроить правила брандмауэра веб-приложения в шлюзе приложений с помощью PowerShell.
services: web-application-firewall
author: vhorne
ms.service: web-application-firewall
ms.date: 11/14/2019
ms.author: victorh
ms.topic: article
ms.openlocfilehash: 55eea15da8c3a10b0421ff1576082d6b42fc7c56
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "74048511"
---
# <a name="customize-web-application-firewall-rules-using-powershell"></a>Настройка правил брандмауэра веб-приложения с помощью PowerShell

Брандмауэр веб-приложения шлюза приложений Azure (WAF) обеспечивает защиту для веб-приложений. с помощью основного набора правил (CRS) открытого проекта безопасности веб-приложений (OWASP). Некоторые правила могут приводить к ложным срабатываниям и блокировке реального трафика. Поэтому шлюз приложений предоставляет возможность настроить правила и группы правил. Дополнительные сведения о конкретных группах правил и правилах см. в разделе [список групп правил и правил брандмауэра веб-приложения](application-gateway-crs-rulegroups-rules.md).

## <a name="view-rule-groups-and-rules"></a>Просмотр правил и групп правил

Ниже приведены примеры кода для просмотра правил и групп правил, которые можно настроить в шлюзе приложений с включенным WAF.

### <a name="view-rule-groups"></a>Просмотр групп правил

В следующем примере показано, как просмотреть группы правил:

```powershell
Get-AzApplicationGatewayAvailableWafRuleSets
```

В результатах далее представлен сокращенный ответ из предыдущего примера.

```
OWASP (Ver. 3.0):

    General:
        Description:

        Rules:
            RuleId     Description
            ------     -----------
            200004     Possible Multipart Unmatched Boundary.

    REQUEST-911-METHOD-ENFORCEMENT:
        Description:

        Rules:
            RuleId     Description
            ------     -----------
            911011     Rule 911011
            911012     Rule 911012
            911100     Method is not allowed by policy
            911013     Rule 911013
            911014     Rule 911014
            911015     Rule 911015
            911016     Rule 911016
            911017     Rule 911017
            911018     Rule 911018

    REQUEST-913-SCANNER-DETECTION:
        Description:

        Rules:
            RuleId     Description
            ------     -----------
            913011     Rule 913011
            913012     Rule 913012
            913100     Found User-Agent associated with security scanner
            913110     Found request header associated with security scanner
            913120     Found request filename/argument associated with security scanner
            913013     Rule 913013
            913014     Rule 913014
            913101     Found User-Agent associated with scripting/generic HTTP client
            913102     Found User-Agent associated with web crawler/bot
            913015     Rule 913015
            913016     Rule 913016
            913017     Rule 913017
            913018     Rule 913018

            ...        ...
```

## <a name="disable-rules"></a>Отключение правил

В следующем примере отключаются правила `911011` и `911012` в шлюзе приложений.

```powershell
$disabledrules=New-AzApplicationGatewayFirewallDisabledRuleGroupConfig -RuleGroupName REQUEST-911-METHOD-ENFORCEMENT -Rules 911011,911012
Set-AzApplicationGatewayWebApplicationFirewallConfiguration -ApplicationGateway $gw -Enabled $true -FirewallMode Detection -RuleSetVersion 3.0 -RuleSetType OWASP -DisabledRuleGroups $disabledrules
Set-AzApplicationGateway -ApplicationGateway $gw
```

## <a name="mandatory-rules"></a>Обязательные правила

В следующем списке содержатся условия, которые приводят к блокировке запроса WAF в режиме предотвращения (в режиме обнаружения они регистрируются как исключения). Их нельзя настроить или отключить.

* Сбой синтаксического анализа текста запроса приводит к блокировке запроса, если проверка тела не отключена (XML, JSON, данные формы).
* Длина данных в тексте запроса (без файлов) превышает заданное ограничение
* Текст запроса (включая файлы) больше ограничения
* В подсистеме WAF произошла внутренняя ошибка

Зависят от CR 3. x:

* Порог входящего индекса аномалий превышает пороговое значение

## <a name="next-steps"></a>Дальнейшие действия

После настройки с отключением правил вы можете узнать, как просматривать журналы WAF. Дополнительные сведения см. в разделе [Диагностика шлюза приложений](../../application-gateway/application-gateway-diagnostics.md#diagnostic-logging).

[fig1]: ./media/application-gateway-customize-waf-rules-portal/1.png
[1]: ./media/application-gateway-customize-waf-rules-portal/figure1.png
[2]: ./media/application-gateway-customize-waf-rules-portal/figure2.png
[3]: ./media/application-gateway-customize-waf-rules-portal/figure3.png
