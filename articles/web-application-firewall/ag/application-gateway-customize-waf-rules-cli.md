---
title: Настройка правил с помощью интерфейса командной строки — брандмауэр веб-приложения Azure
description: В этой статье содержатся сведения о том, как настроить правила брандмауэра веб-приложения в шлюзе приложений с помощью Azure CLI.
services: web-application-firewall
author: vhorne
ms.service: web-application-firewall
ms.date: 11/14/2019
ms.author: victorh
ms.topic: article
ms.openlocfilehash: 8e8aaa9458619bc937c5bb11c450f3197b92f451
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "74048533"
---
# <a name="customize-web-application-firewall-rules-using-the-azure-cli"></a>Настройка правил брандмауэра веб-приложения с помощью Azure CLI

Брандмауэр веб-приложения шлюза приложений Azure (WAF) обеспечивает защиту для веб-приложений. с помощью основного набора правил (CRS) открытого проекта безопасности веб-приложений (OWASP). Некоторые правила могут приводить к ложным срабатываниям и блокировке реального трафика. Поэтому шлюз приложений предоставляет возможность настроить правила и группы правил. Дополнительные сведения о конкретных группах правил и правилах см. в разделе [список групп правил и правил брандмауэра веб-приложения](application-gateway-crs-rulegroups-rules.md).

## <a name="view-rule-groups-and-rules"></a>Просмотр правил и групп правил

Ниже приведены примеры кода для просмотра правил и групп правил, которые можно настроить.

### <a name="view-rule-groups"></a>Просмотр групп правил

В следующем примере показано, как просмотреть группы правил:

```azurecli-interactive
az network application-gateway waf-config list-rule-sets --type OWASP
```

В результатах далее представлен сокращенный ответ из предыдущего примера.

```json
[
  {
    "id": "/subscriptions//resourceGroups//providers/Microsoft.Network/applicationGatewayAvailableWafRuleSets/",
    "location": null,
    "name": "OWASP_3.0",
    "provisioningState": "Succeeded",
    "resourceGroup": "",
    "ruleGroups": [
      {
        "description": "",
        "ruleGroupName": "REQUEST-910-IP-REPUTATION",
        "rules": null
      },
      ...
    ],
    "ruleSetType": "OWASP",
    "ruleSetVersion": "3.0",
    "tags": null,
    "type": "Microsoft.Network/applicationGatewayAvailableWafRuleSets"
  },
  {
    "id": "/subscriptions//resourceGroups//providers/Microsoft.Network/applicationGatewayAvailableWafRuleSets/",
    "location": null,
    "name": "OWASP_2.2.9",
    "provisioningState": "Succeeded",
    "resourceGroup": "",
   "ruleGroups": [
      {
        "description": "",
        "ruleGroupName": "crs_20_protocol_violations",
        "rules": null
      },
      ...
    ],
    "ruleSetType": "OWASP",
    "ruleSetVersion": "2.2.9",
    "tags": null,
    "type": "Microsoft.Network/applicationGatewayAvailableWafRuleSets"
  }
]
```

### <a name="view-rules-in-a-rule-group"></a>Просмотр правил в группе правил

В примере ниже показано, как просмотреть правила в указанной группе правил.

```azurecli-interactive
az network application-gateway waf-config list-rule-sets --group "REQUEST-910-IP-REPUTATION"
```

В результатах далее представлен сокращенный ответ из предыдущего примера.

```json
[
  {
    "id": "/subscriptions//resourceGroups//providers/Microsoft.Network/applicationGatewayAvailableWafRuleSets/",
    "location": null,
    "name": "OWASP_3.0",
    "provisioningState": "Succeeded",
    "resourceGroup": "",
    "ruleGroups": [
      {
        "description": "",
        "ruleGroupName": "REQUEST-910-IP-REPUTATION",
        "rules": [
          {
            "description": "Rule 910011",
            "ruleId": 910011
          },
          ...
        ]
      }
    ],
    "ruleSetType": "OWASP",
    "ruleSetVersion": "3.0",
    "tags": null,
    "type": "Microsoft.Network/applicationGatewayAvailableWafRuleSets"
  }
]
```

## <a name="disable-rules"></a>Отключение правил

В следующем примере отключаются правила `910018` и `910017` в шлюзе приложений.

```azurecli-interactive
az network application-gateway waf-config set --resource-group AdatumAppGatewayRG --gateway-name AdatumAppGateway --enabled true --rule-set-version 3.0 --disabled-rules 910018 910017
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

После настройки с отключением правил вы можете узнать, как просматривать журналы WAF. Дополнительные сведения см. в разделе [Журналы диагностики](../../application-gateway/application-gateway-diagnostics.md#diagnostic-logging).

[fig1]: ./media/application-gateway-customize-waf-rules-portal/1.png
[1]: ./media/application-gateway-customize-waf-rules-portal/figure1.png
[2]: ./media/application-gateway-customize-waf-rules-portal/figure2.png
[3]: ./media/application-gateway-customize-waf-rules-portal/figure3.png
