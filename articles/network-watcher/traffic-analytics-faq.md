---
title: Вопросы и ответы об Аналитике трафика Azure | Документация Майкрософт
description: Ответы на некоторые вопросы об Аналитике трафика Azure.
services: network-watcher
documentationcenter: na
author: damendo
ms.service: network-watcher
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 01/04/2021
ms.author: damendo
ms.openlocfilehash: f4d97b81bf8f11b8380f04dcbfdb72bd658805ab
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100579249"
---
# <a name="traffic-analytics-frequently-asked-questions"></a>Часто задаваемые вопросы по Аналитике трафика Azure

В этой статье приведены многие из наиболее часто задаваемых вопросов о решении "Аналитика трафика" в службе "Наблюдатель за сетями Azure".


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="what-are-the-prerequisites-to-use-traffic-analytics"></a>Что необходимо для работы с Аналитикой трафика?

Для работы с Аналитикой трафика требуются:

- подписка с Наблюдателем за сетями;
- журналы потоков групп безопасности сети (NSG), мониторинг которых необходимо проводить;
- учетная запись службы хранилища Azure для хранения необработанных журналов потоков;
- рабочая область Azure Log Analytics с доступом для чтения и записи.

Для включения решения "Аналитика трафика" ваша учетная запись должна соответствовать одному из следующих условий:

- Ваша учетная запись должна иметь одну из следующих ролей Azure в области подписки: владелец, участник, читатель или участник сети.
- Если вашей учетной записи не назначена ни одна из упомянутых выше ролей, ей должна быть назначена пользовательская роль, предоставляющая разрешения на уровне подписки:
            
    - Microsoft.Network/applicationGateways/read
    - Microsoft.Network/connections/read
    - Microsoft.Network/loadBalancers/read 
    - Microsoft.Network/localNetworkGateways/read 
    - Microsoft.Network/networkInterfaces/read 
    - Microsoft.Network/networkSecurityGroups/read 
    - Microsoft.Network/publicIPAddresses/read
    - Microsoft.Network/routeTables/read
    - Microsoft.Network/virtualNetworkGateways/read 
    - Microsoft.Network/virtualNetworks/read
        
Чтобы проверить роли, назначенные пользователю для подписки:

1. Войдите в Azure с помощью **Login-азаккаунт**. 

2. Выберите нужную подписку с помощью команды **SELECT-азсубскриптион**. 

3. Чтобы получить список всех ролей, назначенных указанному пользователю, используйте  **Get-азролеассигнмент-SignInName [адрес электронной почты пользователя]-инклудеклассикадминистраторс**. 

Если вы не видите выходные данные, обратитесь к соответствующему администратору подписки, чтобы получить доступ к запуску команд. Дополнительные сведения см. в статье [Добавление и удаление назначений ролей Azure с помощью Azure PowerShell](../role-based-access-control/role-assignments-powershell.md).


## <a name="in-which-azure-regions-is-traffic-analytics-available"></a>В каких регионах Azure доступна Аналитика трафика?

Можно использовать аналитику трафика для групп безопасности сети в любом из следующих поддерживаемых регионов:
- Центральная Канада
- центрально-западная часть США
- Восточная часть США
- восточная часть США 2
- Центрально-северная часть США
- Центрально-южная часть США
- Центральная часть США
- западная часть США
- западная часть США 2
- Центральная Франция
- Западная Европа
- Северная Европа
- Южная Бразилия
- западная часть Соединенного Королевства
- южная часть Соединенного Королевства
- Восточная Австралия
- Юго-Восточная часть Австралии 
- Восточная Азия
- Юго-Восточная Азия
- Республика Корея, центральный регион
- Центральная Индия
- Южная Индия
- Восточная Япония
- Западная Япония
- US Gov (Вирджиния)
- Восточный Китай 2

Рабочая область Log Analytics должна присутствовать в следующих регионах:
- Центральная Канада
- центрально-западная часть США
- Восточная часть США
- восточная часть США 2
- Центрально-северная часть США
- Центрально-южная часть США
- Центральная часть США
- западная часть США
- западная часть США 2
- Центральная Франция
- Западная Европа
- Северная Европа
- западная часть Соединенного Королевства
- южная часть Соединенного Королевства
- Восточная Австралия
- Юго-Восточная часть Австралии
- Восточная Азия
- Юго-Восточная Азия 
- Республика Корея, центральный регион
- Центральная Индия
- Восточная Япония
- US Gov (Вирджиния)
- Восточный Китай 2

## <a name="can-the-nsgs-i-enable-flow-logs-for-be-in-different-regions-than-my-workspace"></a>Возможен ли сценарий, когда регион NSG, для которой записываются журналы потоков, и регион рабочей области отличаются?

Да, группа безопасности сети может располагаться не в том же регионе, что рабочая область Log Analytics.

## <a name="can-multiple-nsgs-be-configured-within-a-single-workspace"></a>Можно ли настроить несколько NSG в одной рабочей области?

Да.

## <a name="can-i-use-an-existing-workspace"></a>Можно ли использовать имеющуюся рабочую область?

Да. При выборе такой рабочей области убедитесь, что она переведена на использование нового языка запросов. Если вы не хотите обновлять имеющуюся рабочую область, необходимо создать другую. Дополнительные сведения о новом языке запросов см. в разделе [Azure Monitor журналы обновление до нового поиска по журналам](../azure-monitor/logs/log-query-overview.md).

## <a name="can-my-azure-storage-account-be-in-one-subscription-and-my-log-analytics-workspace-be-in-a-different-subscription"></a>Возможен ли сценарий, когда учетная запись службы хранилища Azure и рабочая область Log Analytics работают на разных подписках?

Да. Ваши учетная запись службы хранилища Azure и рабочая область Log Analytics могут работать на разных подписках.

## <a name="can-i-store-raw-logs-in-a-different-subscription"></a>Можно ли хранить необработанные журналы в другой подписке?

Да. Вы можете настроить отправку журналов потоков NSG в учетную запись хранения, расположенную в другой подписке, при условии, что у вас есть соответствующие привилегии, и что учетная запись хранения находится в том же регионе, что и NSG. NSG и Целевая учетная запись хранения также должны совместно использовать один и тот же клиент Azure Active Directory.

## <a name="what-if-i-cant-configure-an-nsg-for-traffic-analytics-due-to-a-not-found-error"></a>Что делать, если я не могу настроить NSG для Аналитики трафика из-за ошибки "Не найдено"?

Выберите поддерживаемый регион. Если выбрать регион, который не поддерживается, вы получите сообщение об ошибке «Не найдено». Поддерживаемые регионы были перечислены ранее в этой статье.

## <a name="what-if-i-am-getting-the-status-failed-to-load-under-the-nsg-flow-logs-page"></a>Что делать, если на странице журналов потоков NSG отображается состояние "Не удалось загрузить"?

Чтобы запись данных о потоках выполнялась правильно, необходимо зарегистрировать поставщик Microsoft.Insights. Если вы не знаете точно, зарегистрирован ли поставщик Microsoft.Insights в вашей подписке, укажите идентификатор вместо заменителя *xxxxx-xxxxx-xxxxxx-xxxx* в команде ниже и выполните такие команды в PowerShell:

```powershell-interactive
**Select-AzSubscription** -SubscriptionId xxxxx-xxxxx-xxxxxx-xxxx
**Register-AzResourceProvider** -ProviderNamespace Microsoft.Insights
```

## <a name="i-have-configured-the-solution-why-am-i-not-seeing-anything-on-the-dashboard"></a>Решение настроено. Почему на панели мониторинга не отображается соответствующее содержимое?

При первой загрузке отображение панели мониторинга может занять до 30 минут. Решение сначала должно выполнить достаточно статистических вычислений для того, чтобы произвести важные аналитические сведения. Затем оно создает отчеты. 

## <a name="what-if-i-get-this-message-we-could-not-find-any-data-in-this-workspace-for-selected-time-interval-try-changing-the-time-interval-or-select-a-different-workspace"></a>Что, если я получу сообщение о том, что не удалось найти данные в рабочей области для выбранного интервала времени и следует попробовать изменить интервал времени или выбрать другую рабочую область?

Есть следующие решения.
- Измените интервал времени на верхней панели.
- Выберите другую рабочую область Log Analytics на верхней панели.
- Попытайтесь открыть Аналитику трафика через 30 минут, если она работает недавно.
    
Если проблемы возникнут снова, обратитесь за помощью на [пользовательском форуме](https://feedback.azure.com/forums/217313-networking?category_id=195844).

## <a name="what-if-i-get-this-message-analyzing-your-nsg-flow-logs-for-the-first-time-this-process-may-take-20-30-minutes-to-complete-check-back-after-some-time-2-if-the-above-step-doesnt-work-and-your-workspace-is-under-the-free-sku-then-check-your-workspace-usage-here-to-validate-over-quota-else-refer-to-faqs-for-further-information"></a>Что, если я получу сообщение о том, что журналы потоков NSG анализируются впервые, это может занять от 20 до 30 минут и следует подождать некоторое время; 2) если способ выше не сработал, а у рабочей области бесплатный номер SKU, следует проверить используемый объем рабочей области, чтобы убедиться, что не превышена квота, или просмотреть раздел часто задаваемых вопросов, чтобы узнать дополнительные сведения?

Это сообщение может появиться, потому что:
- Аналитика трафика была включено недавно, статистических вычислений может быть недостаточно, чтобы получить важные аналитические сведения.
- Вы используете бесплатную версию рабочей области Log Analytics и превысили квоту. Возможно, вам придется использовать рабочую область с большей емкостью.
    
Если проблемы возникнут снова, обратитесь за помощью на [пользовательском форуме](https://feedback.azure.com/forums/217313-networking?category_id=195844).
    
## <a name="what-if-i-get-this-message-looks-like-we-have-resources-data-topology-and-no-flows-information-meanwhile-click-here-to-see-resources-data-and-refer-to-faqs-for-further-information"></a>Что, если я получу сообщение о том, что, возможно, есть данные о ресурсах (топология) и нет информации о потоках, и следует щелкнуть ссылку, чтобы просмотреть данные ресурса, и ознакомиться с часто задаваемым вопросами для получения дополнительных сведений?

Если на панели мониторинга есть сведения о ресурсах, но нет статистических данных о потоках, возможно, нет потоков связи между ресурсами. Подождите 60 мин и проверьте состояние снова. Если проблема существует и вы уверены в наличии потоков связи между всеми ресурсами, обратитесь за помощью на [пользовательском форуме](https://feedback.azure.com/forums/217313-networking?category_id=195844).

## <a name="can-i-configure-traffic-analytics-using-powershell-or-an-azure-resource-manager-template-or-client"></a>Можно ли настроить решение "Аналитика трафика" с помощью PowerShell, шаблона или клиента Azure Resource Manager?

Вы можете настроить решение "Аналитика трафика" с помощью Windows PowerShell, начиная с версии 6.2.1. Сведения о настройке ведения журнала потоков и анализа трафика для определенного NSG с помощью командлета Set см. в разделе [Set-азнетворкватчерконфигфловлог](/powershell/module/az.network/set-aznetworkwatcherconfigflowlog). Сведения о получении журнала потоков и состояния аналитики трафика для определенного NSG см. в разделе [Get-азнетворкватчерфловлогстатус](/powershell/module/az.network/get-aznetworkwatcherflowlogstatus).

В настоящее время вы не можете использовать шаблон Azure Resource Manager для настройки решения "Аналитика трафика".

Чтобы настроить решение "Аналитика трафика" с помощью клиента Azure Resource Manager, см. следующие примеры.

**Пример командлета Set:**
```
#Requestbody parameters
$TAtargetUri ="/subscriptions/<NSG subscription id>/resourceGroups/<NSG resource group name>/providers/Microsoft.Network/networkSecurityGroups/<name of NSG>"
$TAstorageId = "/subscriptions/<storage subscription id>/resourcegroups/<storage resource group name> /providers/microsoft.storage/storageaccounts/<storage account name>"
$networkWatcherResourceGroupName = "<network watcher resource group name>"
$networkWatcherName = "<network watcher name>"

$requestBody = 
@"
{
    'targetResourceId': '${TAtargetUri}',
    'properties': 
    {
        'storageId': '${TAstorageId}',
        'enabled': '<true to enable flow log or false to disable flow log>',
        'retentionPolicy' : 
        {
            days: <enter number of days like to retail flow logs in storage account>,
            enabled: <true to enable retention or false to disable retention>
        }
    },
    'flowAnalyticsConfiguration':
    {
                'networkWatcherFlowAnalyticsConfiguration':
      {
        'enabled':,<true to enable traffic analytics or false to disable traffic analytics>
        'workspaceId':'bbbbbbbb-bbbb-bbbb-bbbb-bbbbbbbbbbbb',
        'workspaceRegion':'<workspace region>',
        'workspaceResourceId':'/subscriptions/<workspace subscription id>/resourcegroups/<workspace resource group name>/providers/microsoft.operationalinsights/workspaces/<workspace name>'
        
      }

    }
}
"@
$apiversion = "2016-09-01"

armclient login
armclient post "https://management.azure.com/subscriptions/<NSG subscription id>/resourceGroups/<network watcher resource group name>/providers/Microsoft.Network/networkWatchers/<network watcher name>/configureFlowlog?api-version=${apiversion}" $requestBody
```
**Пример командлета Get:**
```
#Requestbody parameters
$TAtargetUri ="/subscriptions/<NSG subscription id>/resourceGroups/<NSG resource group name>/providers/Microsoft.Network/networkSecurityGroups/<NSG name>"


$requestBody = 
@"
{
    'targetResourceId': '${TAtargetUri}'
}
“@


armclient login
armclient post "https://management.azure.com/subscriptions/<NSG subscription id>/resourceGroups/<network watcher resource group name>/providers/Microsoft.Network/networkWatchers/<network watcher name>/queryFlowLogStatus?api-version=${apiversion}" $requestBody
```


## <a name="how-is-traffic-analytics-priced"></a>Как образуются цены на Аналитику трафика?

Аналитика трафика измеряется на основе обработки данных журнала потоков службой и сохранении полученных расширенных журналов в рабочей области Log Analytics. 

Например, согласно [тарифному плану](https://azure.microsoft.com/pricing/details/network-watcher/) для региона "Центрально-западная часть США", если объем данных в журналах потоков, хранящихся в учетной записи хранения и обрабатываемых решением "Аналитика трафика", составляет 10 ГБ, а объем улучшенных журналов, принятых в рабочей области Log Analytics, — 1 ГБ, то применимый тариф рассчитывается так: 10 x 2,3 долл. США + 1 x 2,76 долл. США = 25,76 долл. США.

## <a name="how-frequently-does-traffic-analytics-process-data"></a>Как часто Аналитика трафика обрабатывать данные?

См. [раздел агрегирование данных](./traffic-analytics-schema.md#data-aggregation) в схеме аналитика трафика и документе агрегирования данных.

## <a name="how-does-traffic-analytics-decide-that-an-ip-is-malicious"></a>Как Аналитика трафика решает, что IP-адрес является вредоносным? 

Аналитика трафика полагается на внутренние системы анализа угроз Майкрософт, чтобы считаться вредоносным. Эти системы используют различные источники телеметрии, такие как продукты и службы Майкрософт, модуль Microsoft Digital Crimes Unit (DCU), центр Microsoft Security Response Center (MSRC) и внешние веб-каналы, а также поверх нее. Некоторые из этих данных являются Microsoft internal. Если известный IP-адрес помечается как вредоносный, создайте запрос в службу поддержки, чтобы узнать подробности.

## <a name="how-can-i-set-alerts-on-traffic-analytics-data"></a>Как настроить оповещения для данных Аналитика трафика?

Аналитика трафика не поддерживает встроенную поддержку предупреждений. Однако, поскольку Аналитика трафика данные хранятся в Log Analytics можно создавать пользовательские запросы и задавать для них оповещения. Выполнены
- Вы можете использовать шортлинк для Log Analytics в Аналитика трафика. 
- Используйте [схему, описанную здесь](traffic-analytics-schema.md) для написания запросов 
- Щелкните "создать правило генерации оповещений", чтобы создать оповещение.
- Сведения о создании оповещения см. в [документации по оповещениям журнала](../azure-monitor/alerts/alerts-log.md) .

## <a name="how-do-i-check-which-vms-are-receiving-most-on-premises-traffic"></a>Разделы справки проверить, какие виртуальные машины получают наибольший объем локального трафика?

```
AzureNetworkAnalytics_CL
| where SubType_s == "FlowLog" and FlowType_s == "S2S" 
| where <Scoping condition>
| mvexpand vm = pack_array(VM1_s, VM2_s) to typeof(string)
| where isnotempty(vm) 
| extend traffic = AllowedInFlows_d + DeniedInFlows_d + AllowedOutFlows_d + DeniedOutFlows_d // For bytes use: | extend traffic = InboundBytes_d + OutboundBytes_d 
| make-series TotalTraffic = sum(traffic) default = 0 on FlowStartTime_t from datetime(<time>) to datetime(<time>) step 1m by vm
| render timechart
```

  Для IP-адресов:

```
AzureNetworkAnalytics_CL
| where SubType_s == "FlowLog" and FlowType_s == "S2S" 
//| where <Scoping condition>
| mvexpand IP = pack_array(SrcIP_s, DestIP_s) to typeof(string)
| where isnotempty(IP) 
| extend traffic = AllowedInFlows_d + DeniedInFlows_d + AllowedOutFlows_d + DeniedOutFlows_d // For bytes use: | extend traffic = InboundBytes_d + OutboundBytes_d 
| make-series TotalTraffic = sum(traffic) default = 0 on FlowStartTime_t from datetime(<time>) to datetime(<time>) step 1m by IP
| render timechart
```

Для времени используйте формат: гггг-мм-дд 00:00:00

## <a name="how-do-i-check-standard-deviation-in-traffic-received-by-my-vms-from-on-premises-machines"></a>Разделы справки проверить стандартное отклонение трафика, полученного из виртуальных машин из локальных компьютеров?

```
AzureNetworkAnalytics_CL
| where SubType_s == "FlowLog" and FlowType_s == "S2S" 
//| where <Scoping condition>
| mvexpand vm = pack_array(VM1_s, VM2_s) to typeof(string)
| where isnotempty(vm) 
| extend traffic = AllowedInFlows_d + DeniedInFlows_d + AllowedOutFlows_d + DeniedOutFlows_d // For bytes use: | extend traffic = InboundBytes_d + utboundBytes_d
| summarize deviation = stdev(traffic)  by vm
```

Для IP-адресов:

```
AzureNetworkAnalytics_CL
| where SubType_s == "FlowLog" and FlowType_s == "S2S" 
//| where <Scoping condition>
| mvexpand IP = pack_array(SrcIP_s, DestIP_s) to typeof(string)
| where isnotempty(IP) 
| extend traffic = AllowedInFlows_d + DeniedInFlows_d + AllowedOutFlows_d + DeniedOutFlows_d // For bytes use: | extend traffic = InboundBytes_d + OutboundBytes_d
| summarize deviation = stdev(traffic)  by IP
```

## <a name="how-do-i-check-which-ports-are-reachable-or-blocked-between-ip-pairs-with-nsg-rules"></a>Разделы справки проверить, какие порты являются доступными (или заблокированными) между парами IP-адресов с правилами NSG?

```
AzureNetworkAnalytics_CL
| where SubType_s == "FlowLog" and TimeGenerated between (startTime .. endTime)
| extend sourceIPs = iif(isempty(SrcIP_s), split(SrcPublicIPs_s, " ") , pack_array(SrcIP_s)),
destIPs = iif(isempty(DestIP_s), split(DestPublicIPs_s," ") , pack_array(DestIP_s))
| mvexpand SourceIp = sourceIPs to typeof(string)
| mvexpand DestIp = destIPs to typeof(string)
| project SourceIp = tostring(split(SourceIp, "|")[0]), DestIp = tostring(split(DestIp, "|")[0]), NSGList_s, NSGRule_s, DestPort_d, L4Protocol_s, FlowStatus_s 
| summarize DestPorts= makeset(DestPort_d) by SourceIp, DestIp, NSGList_s, NSGRule_s, L4Protocol_s, FlowStatus_s
```

## <a name="how-can-i-navigate-by-using-the-keyboard-in-the-geo-map-view"></a>Как перемещаться в представлении географической карты с помощью клавиатуры?

На странице географической карты есть два основных раздела:
    
- **Баннер**: баннер в верхней части географической схемы содержит кнопки для выбора фильтров распределения трафика (например, развертывание, трафик из стран и регионов, а также вредоносный). При нажатии кнопки соответствующий фильтр применяется на карте. Например, если нажать кнопку "Активный", карта выделит активные центры обработки данных в развертывании.
- **Map**: под баннером в разделе Map показано распределение трафика между центрами обработки данных Azure и странами/регионами.
    
### <a name="keyboard-navigation-on-the-banner"></a>Навигация по баннеру с помощью клавиатуры
    
- По умолчанию на странице географической карты на баннере нажата кнопка фильтра Azure DC.
- Чтобы переместиться к другому фильтру, используйте клавишу `Tab` или `Right arrow`. Чтобы вернуться к предыдущему фильтру, используйте клавишу `Shift+Tab` или `Left arrow`. Порядок перемещения вперед: слева направо и сверху вниз.
- Чтобы применить выбранный фильтр, нажмите клавишу `Enter` или клавишу со стрелкой `Down`. В зависимости от развертывания и выбранного фильтра в разделе карты будут выделены один или несколько узлов.
- Чтобы перейти из раздела "Баннер" в "Карты" и наоборот, нажмите клавишу `Ctrl+F6`.
        
### <a name="keyboard-navigation-on-the-map"></a>Навигация по карте с помощью клавиатуры
    
- Чтобы переместиться от одного выделенного узла (**Azure datacenter** (Центр обработки данных Azure) или **Страна/регион**) в представлении карты, выберите любой фильтр на баннере и нажмите клавиши `Ctrl+F6`.
- Чтобы перейти к другим выделенным узлам на карте, используйте клавишу `Tab` или `Right arrow` для перемещения вперед. Используйте клавиши `Shift+Tab` и `Left arrow`, чтобы переместиться назад.
- Чтобы выбрать любой выделяемый узел карты, используйте клавишу `Enter` или `Down arrow`.
- При выборе любого такого узла фокус перемещается на **панель элементов "Информация"** для этого узла. По умолчанию фокус перемещается на кнопку закрытия **панели элементов "Информация"**. Для дальнейшего перемещения в представлении **панели** используйте клавиши `Right arrow` и `Left arrow`, чтобы переходить вперед и назад соответственно. Нажатие клавиши `Enter` действует так же, как кнопка с фокусом на **панели элементов "Информация"**.
- Если нажать клавишу `Tab`, когда фокус на **панели элементов "Информация"**, фокус переместится к конечным точкам, которые находятся на том же континенте, что и выбранный узел. Для перемещения между этими конечными точками можно использовать клавиши `Right arrow` и `Left arrow`.
- Для перемещения к конечным точкам или кластеру континента другого потока используйте клавишу `Tab`, чтобы переходить вперед, и клавиши `Shift+Tab`, чтобы перейти назад.
- Чтобы выделить конечные точки, нажмите клавишу `Enter` или `Down`, когда фокус находится на **кластере континента**. Для перехода между конечными точками и к кнопке закрытия на панели "Информация" для кластера континента используйте клавишу `Right arrow` или `Left arrow`, чтобы перемещаться вперед и назад соответственно. Чтобы перейти к строке подключения выбранного узла к строке конечной точки, нажмите клавиши `Shift+L` для любой такой точки. Нажмите клавиши `Shift+L` еще раз, чтобы перейти к выбранной конечной точке.
        
### <a name="keyboard-navigation-at-any-stage"></a>Навигация с помощью клавиатуры на любом этапе
    
- Клавиша `Esc` позволяет свернуть развернутый элемент.
- Клавиша `Up-arrow` работает так же, как `Esc`. Клавиша `Down arrow` работает так же, как `Enter`.
- Клавиши `Shift+Plus` позволяют увеличить масштаб, а `Shift+Minus` — уменьшить его.

## <a name="how-can-i-navigate-by-using-the-keyboard-in-the-virtual-network-topology-view"></a>Как перемещаться в представлении топологии виртуальной сети с помощью клавиатуры?

Страница топологии виртуальных сетей содержит два основных раздела:
    
- **Баннер**. Баннер в верхней части топологии виртуальных сетей предоставляет кнопки для выбора фильтров распределения трафика (например, подключенные виртуальные сети, отключенные виртуальные сети и общедоступные IP-адреса). При нажатии кнопки соответствующий фильтр применяется в топологии. Например, если нажать кнопку "Активная", топология выделит активные виртуальные сети в развертывании.
- **Топология**. Под баннером раздел топологии показывает распределение трафика между виртуальными сетями.
    
### <a name="keyboard-navigation-on-the-banner"></a>Навигация по баннеру с помощью клавиатуры
    
- По умолчанию на странице топологии виртуальных сетей для баннера выбран фильтр "Подключенные виртуальные сети".
- Чтобы переместиться к другому фильтру, используйте клавишу `Tab`. Чтобы вернуться к предыдущему, используйте клавиши `Shift+Tab`. Порядок перемещения вперед: слева направо и сверху вниз.
- Чтобы применить выбранный фильтр, нажмите клавишу `Enter`. В зависимости от развертывания и выбранного фильтра в разделе топологии будут выделены один или несколько узлов (виртуальная сеть).
- Чтобы переключиться между баннером и топологией, нажмите клавишу `Ctrl+F6`.
        
### <a name="keyboard-navigation-on-the-topology"></a>Навигация по топологии с помощью клавиатуры
    
- Чтобы переместиться от одного выделенного узла (**Виртуальная сеть**) в представлении топологии, выберите любой фильтр на баннере и нажмите клавиши `Ctrl+F6`.
- Чтобы перейти к другим выделенным узлам в представлении топологии, используйте клавиши `Shift+Right arrow` для перемещения вперед. 
- На выделенных узлах фокус перемещается на **панель элементов "Информация"** для этого узла. По умолчанию фокус перемещается на кнопку **Дополнительные сведения****панели элементов "Информация"**. Для дальнейшего перемещения в представлении **панели** используйте клавиши `Right arrow` и `Left arrow`, чтобы переходить вперед и назад соответственно. Нажатие клавиши `Enter` действует так же, как кнопка с фокусом на **панели элементов "Информация"**.
- При выборе любых подобных узлов можно просмотреть все их подключения, одно за одним, нажимая клавиши `Shift+Left arrow`. Фокус перемещается на **панель элементов "Информация"** этого подключения. В любой момент фокус можно сдвинуть обратно на узел, нажав клавиши `Shift+Right arrow` еще раз.
    

## <a name="how-can-i-navigate-by-using-the-keyboard-in-the-subnet-topology-view"></a>Как перемещаться в представлении топологии подсети с помощью клавиатуры?

Страница топологии виртуальных подсетей содержит два основных раздела:
    
- **Баннер**. Баннер в верхней части топологии виртуальных подсетей предоставляет кнопки для выбора фильтров распределения трафика (например, подсетей "Активная", "Средняя", "Шлюз"). При нажатии кнопки соответствующий фильтр применяется в топологии. Например, если нажать кнопку "Активная", топология выделит активную виртуальную подсеть в развертывании.
- **Топология**. Под баннером раздел топологии показывает распределение трафика между виртуальными подсетями.
    
### <a name="keyboard-navigation-on-the-banner"></a>Навигация по баннеру с помощью клавиатуры
    
- По умолчанию на странице топологии виртуальных подсетей для баннера выбран фильтр "Подсети".
- Чтобы переместиться к другому фильтру, используйте клавишу `Tab`. Чтобы вернуться к предыдущему, используйте клавиши `Shift+Tab`. Порядок перемещения вперед: слева направо и сверху вниз.
- Чтобы применить выбранный фильтр, нажмите клавишу `Enter`. В зависимости от развертывания и выбранного фильтра (подсети) в разделе топологии будут выделены один или несколько узлов.
- Чтобы переключиться между баннером и топологией, нажмите клавишу `Ctrl+F6`.
        
### <a name="keyboard-navigation-on-the-topology"></a>Навигация по топологии с помощью клавиатуры
    
- Чтобы переместиться от одного выделенного узла (**Подсеть**) в представлении топологии, выберите любой фильтр на баннере и нажмите клавиши `Ctrl+F6`.
- Чтобы перейти к другим выделенным узлам в представлении топологии, используйте клавиши `Shift+Right arrow` для перемещения вперед. 
- На выделенных узлах фокус перемещается на **панель элементов "Информация"** для этого узла. По умолчанию фокус перемещается на кнопку **Дополнительные сведения****панели элементов "Информация"**. Для дальнейшего перемещения в представлении **панели** используйте клавиши `Right arrow` и `Left arrow`, чтобы переходить вперед и назад соответственно. Нажатие клавиши `Enter` действует так же, как кнопка с фокусом на **панели элементов "Информация"**.
- При выборе любых подобных узлов можно просмотреть все их подключения, одно за одним, нажимая клавиши `Shift+Left arrow`. Фокус перемещается на **панель элементов "Информация"** этого подключения. В любой момент фокус можно сдвинуть обратно на узел, нажав клавиши `Shift+Right arrow` еще раз.

## <a name="are-classic-nsgs-supported"></a>Поддерживаются ли классические группы безопасности сети?
Нет, Аналитика трафика не поддерживает классическую NSG. Рекомендуется перенести ресурсы IaaS из классической модели в Azure Resource Manager как классические ресурсы будут считаться [устаревшими](../virtual-machines/classic-vm-deprecation.md). Сведения [о миграции](../virtual-machines/migration-classic-resource-manager-overview.md)см. в этой статье.