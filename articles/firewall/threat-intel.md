---
title: Фильтрация на основе аналитики угроз в Брандмауэре Azure
description: Фильтрация на основе Microsoft Threat Intelligence может быть включена в брандмауэре с целю создания оповещений и запрета трафика, поступающего с известных вредоносных IP-адресов и доменов, а также передающегося на них.
services: firewall
author: vhorne
ms.service: firewall
ms.topic: article
ms.date: 05/12/2020
ms.author: victorh
ms.openlocfilehash: 77e36d5ab98f1177b5a0b89ee10b3e6bd22560c2
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98791465"
---
# <a name="azure-firewall-threat-intelligence-based-filtering"></a>Фильтрация на основе аналитики угроз в Брандмауэре Azure

Фильтрация на основе Microsoft Threat Intelligence может быть включена в брандмауэре с целю создания оповещений и запрета трафика, поступающего с известных вредоносных IP-адресов и доменов, а также передающегося на них. IP-адреса и домены также передаются из канала Microsoft Threat Intelligence. [Intelligent Security Graph](https://www.microsoft.com/security/operations/intelligence) поддерживает Microsoft Threat Intelligence и используется во множестве служб, включая Центр безопасности Azure.<br>
<br>

:::image type="content" source="media/threat-intel/firewall-threat.png" alt-text="Аналитика угроз в брандмауэре" border="false":::

Если включена фильтрация на основе аналитики угроз, то связанные правила обрабатываются прежде любых правил NAT, правил сети или правил приложения.

Можно выбрать только ведение журнала оповещений при активации правила или выбрать режим оповещения и запрета.

По умолчанию фильтрация на основе аналитики угроз работает в режиме оповещения. Вы не сможете отключить эту функцию или изменить режим, пока соответствующий раздел интерфейса портала не станет доступен в вашем регионе.

:::image type="content" source="media/threat-intel/threat-intel-ui.png" alt-text="Интерфейс портала для фильтрации на основе аналитики угроз":::

## <a name="logs"></a>Журналы

В следующем фрагменте журнала показано активированное правило.

```
{
    "category": "AzureFirewallNetworkRule",
    "time": "2018-04-16T23:45:04.8295030Z",
    "resourceId": "/SUBSCRIPTIONS/{subscriptionId}/RESOURCEGROUPS/{resourceGroupName}/PROVIDERS/MICROSOFT.NETWORK/AZUREFIREWALLS/{resourceName}",
    "operationName": "AzureFirewallThreatIntelLog",
    "properties": {
         "msg": "HTTP request from 10.0.0.5:54074 to somemaliciousdomain.com:80. Action: Alert. ThreatIntel: Bot Networks"
    }
}
```

## <a name="testing"></a>Тестирование

- **Тестирование исходящего трафика**. Оповещения об исходящем трафике должны появляться редко, так как они означают, что ваша среда была скомпрометирована. Чтобы проверить работоспособность оповещений об исходящем трафике, было создано тестовое полное доменное имя, которое активирует оповещение. Используйте **testmaliciousdomain.eastus.cloudapp.azure.com** для тестирования исходящего трафика.

- **Тестирование входящего трафика**. Вы можете видеть оповещения о входящем трафике, если в брандмауэре настроены правила DNAT. Это происходит, даже если в правиле DNAT разрешены только определенные источники, а трафик из других источников отклоняется. Брандмауэр Azure не оповещает о всех известных сканерах портов. Он оповещает только об известных сканерах, которые замечены во вредоносных действиях.

## <a name="next-steps"></a>Дальнейшие действия

- Ознакомьтесь с [примерами запросов Log Analytics для Брандмауэра Azure](./firewall-workbook.md).
- Узнайте, как [развернуть и настроить Брандмауэр Azure](tutorial-firewall-deploy-portal.md).
- Изучите [аналитический отчет Microsoft Security](https://www.microsoft.com/en-us/security/operations/security-intelligence-report).