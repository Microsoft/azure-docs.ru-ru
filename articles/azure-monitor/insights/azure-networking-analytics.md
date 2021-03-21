---
title: Решение для анализа сетей Azure в Azure Monitor | Документация Майкрософт
description: Решение для анализа сетей Azure можно использовать в Azure Monitor для просмотра журналов группы безопасности сети Azure и Шлюза приложений Azure.
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 06/21/2018
ms.openlocfilehash: b8b03378e82810bc2b9680805bacf8360f322a94
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101708141"
---
# <a name="azure-networking-monitoring-solutions-in-azure-monitor"></a>Решения для мониторинга сетей Azure в Azure Monitor

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

В Azure Monitor предлагаются следующие решения для мониторинга сетей.
* Монитор производительности сети (NPM):
    * отслеживание работоспособности сети.
* Анализ шлюзов приложений Azure для проверки:
    * журналы шлюза приложений Azure;
    * метрику шлюза приложений Azure.
* Решения для отслеживания и аудита сетевой активности в облачной сети.
    * [Анализ трафика](../../networking/network-monitoring-overview.md#traffic-analytics). 
    * Анализ групп безопасности сети Azure

## <a name="network-performance-monitor-npm"></a>Монитор производительности сети

Решение по управлению [Монитор производительности сети](../../networking/network-monitoring-overview.md) — это решение для мониторинга сети, которое отслеживает работоспособность и доступность сетей.  Оно используется для отслеживания подключения между следующими ресурсами:

* общедоступное облако и локальная среда;
* центры обработки данных и расположения пользователей (филиалы);
* подсети, в которых размещены различные уровни многоуровневого приложения.

Дополнительную информацию см. в статье [Решение монитора производительности сети в Azure Log Analytics](../../networking/network-monitoring-overview.md).

## <a name="network-security-group-analytics"></a>Анализ групп безопасности сети

1. Добавьте решение по управлению в Azure Monitor.
2. Включите отправку данных диагностики в рабочую область Log Analytics в Azure Monitor. Необязательно записывать журналы в хранилище BLOB-объектов Azure.

Если журналы диагностики не включены, колонки панели мониторинга для этого ресурса будут пустыми и отобразится сообщение об ошибке.

## <a name="azure-application-gateway-analytics"></a>Анализ шлюзов приложений Azure

1. Включите отправку данных диагностики в рабочую область Log Analytics в Azure Monitor.
2. Используйте подробную сводку для ресурса с помощью шаблона книги для шлюза приложений.

Если журналы диагностики не включены для шлюза приложений, в книге будут заполнены только данные метрик по умолчанию.


> [!NOTE]
> В январе 2017 г. был изменен поддерживаемый способ отправки журналов из Шлюзов приложений и групп безопасности сети в рабочую область Log Analytics. Если отобразится устаревшее решение **Анализ сетевой активности Azure (не рекомендуется)** , то выполните действия, описанные в разделе [Миграция из устаревшего решения для анализа сетевой активности](#migrating-from-the-old-networking-analytics-solution).
>
>

## <a name="review-azure-networking-data-collection-details"></a>Просмотр сведений о сборе данных о сетях Azure
Решения для управления анализом шлюзов приложений и групп безопасности сети Azure собирают журналы диагностики непосредственно из шлюзов приложений и групп безопасности сети Azure. Необязательно записывать журналы в хранилище BLOB-объектов Azure. Для сбора данных агенты не требуются.

В следующей таблице приведены методы сбора данных и другие сведения о сборе данных для анализа шлюзов приложений и групп безопасности сети Azure.

| Платформа | Direct Agent | Агент Systems Center Operations Manager | Azure | Нужен ли Operations Manager? | Отправка данных агента Operations Manager через группу управления | Частота сбора |
| --- | --- | --- | --- | --- | --- | --- |
| Azure |  |  |&#8226; |  |  |при входе |


### <a name="enable-azure-application-gateway-diagnostics-in-the-portal"></a>Включение диагностики шлюза приложений Azure на портале

1. На портале Azure перейдите к ресурсу Шлюза приложений, который нужно отслеживать.
2. Выберите *параметры диагностики* , чтобы открыть следующую страницу.

   ![Снимок экрана с конфигурацией параметров диагностики для ресурса шлюза приложений.](media/azure-networking-analytics/diagnostic-settings-1.png)

   [![Снимок экрана страницы для настройки параметров диагностики.](media/azure-networking-analytics/diagnostic-settings-2.png)](media/azure-networking-analytics/application-gateway-diagnostics-2.png#lightbox)

5. Установите флажок *Отправить в Log Analytics*.
6. Выберите существующую рабочую область Log Analytics или создайте рабочую область.
7. В разделе **Журнал** установите флажки для всех собираемых типов журналов.
8. Щелкните *Сохранить*, чтобы включить ведение журнала диагностики в Azure Monitor.

#### <a name="enable-azure-network-diagnostics-using-powershell"></a>Включение диагностики сети Azure с помощью PowerShell

В следующем примере скрипта PowerShell показано, как включить ведение журналов диагностики для шлюзов приложений.

```powershell
$workspaceId = "/subscriptions/d2e37fee-1234-40b2-5678-0b2199de3b50/resourcegroups/oi-default-east-us/providers/microsoft.operationalinsights/workspaces/rollingbaskets"

$gateway = Get-AzApplicationGateway -Name 'ContosoGateway'

Set-AzDiagnosticSetting -ResourceId $gateway.ResourceId  -WorkspaceId $workspaceId -Enabled $true
```

#### <a name="accessing-azure-application-gateway-analytics-via-azure-monitor-network-insights"></a>Доступ к аналитике шлюза приложений Azure с помощью Azure Monitor Network Insights

Доступ к Application Insights можно получить с помощью вкладки "аналитика" в ресурсе шлюза приложений.

![Снимок экрана: аналитика шлюза приложений ](media/azure-networking-analytics/azure-appgw-insights.png
)

На вкладке "просмотреть подробные метрики" откроется предварительно заполненная книга со сводкой данных из шлюза приложений.

[![Снимок экрана книги шлюза приложений](media/azure-networking-analytics/azure-appgw-workbook.png)](media/azure-networking-analytics/application-gateway-workbook.png#lightbox)

### <a name="new-capabilities-with-azure-monitor-network-insights-workbook"></a>Новые возможности в книге Azure Monitor Network Insights

> [!NOTE]
> Нет дополнительных затрат, связанных с книгой Azure Monitor Insights. В Log Analytics рабочей области будет взиматься плата за использование.

Книга Network Insights позволяет воспользоваться преимуществами новейших возможностей Azure Monitor и Log Analytics включая:

* Централизованная консоль для мониторинга и устранения неполадок с данными [метрик](../insights/network-insights-overview.md#resource-health-and-metrics) и журналов.

* Гибкое полотно для поддержки создания пользовательских полнофункциональных [визуализаций](../visualize/workbooks-overview.md#visualizations).

* Возможность использования и [совместного использования шаблонов книг](../visualize/workbooks-overview.md#workbooks-versus-workbook-templates) с более широкими сообществом.

Дополнительные сведения о возможностях нового решения для книги извлечь [книги — обзор](../visualize/workbooks-overview.md)

## <a name="migrating-from-azure-gateway-analytics-solution-to-azure-monitor-workbooks"></a>Миграция из решения аналитики шлюза Azure в Azure Monitor книги

> [!NOTE]
> Книга Azure Monitor Network Insights — это рекомендуемое решение для доступа к метрикам и log Analytics для ресурсов шлюза приложений.

1. Убедитесь [, что параметры диагностики включены](#enable-azure-application-gateway-diagnostics-in-the-portal) для хранения журналов в log Analytics рабочей области. Если она уже настроена, то Azure Monitor книга "аналитика сети" сможет использовать данные из одного и того же расположения, и никаких дополнительных изменений не потребуется.

> [!NOTE]
> Все прошлые данные уже доступны в книге из параметров диагностики точек изначально были включены. Перенос данных не требуется.

2. Получите доступ к [книге Insights по умолчанию](#accessing-azure-application-gateway-analytics-via-azure-monitor-network-insights) для ресурса шлюза приложений. Все существующие аналитические данные, поддерживаемые решением аналитики шлюза приложений, будут уже присутствовать в книге. Это можно расширить, добавив пользовательские [визуализации](../visualize/workbooks-overview.md#visualizations) на основе метрик & данных журнала.

3. После того как вы сможете просмотреть все данные метрик и журналов, чтобы очистить решение аналитики шлюза Azure из рабочей области, можно удалить решение на странице ресурсов решения.

[![Снимок экрана с параметром "Удалить" для решения "анализ шлюза приложений Azure".](media/azure-networking-analytics/azure-appgw-analytics-delete.png)](media/azure-networking-analytics/application-gateway-analytics-delete.png#lightbox)

## <a name="azure-network-security-group-analytics-solution-in-azure-monitor"></a>Решение "Аналитика групп безопасности сетей Azure" в Azure Monitor

![Символ "Аналитика групп безопасности сетей Azure"](media/azure-networking-analytics/azure-analytics-symbol.png)

> [!NOTE]
> Решение для анализа групп безопасности сети переходит на поддержку сообщества, так как оно было заменено на [Аналитику трафика](../../network-watcher/traffic-analytics.md).
> - Это решение теперь доступно в [Шаблонах быстрого запуска Azure](https://azure.microsoft.com/resources/templates/oms-azurensg-solution/) и скоро перестанет быть доступным в Azure Marketplace.
> - Для существующих клиентов, которые уже добавили решение в свою рабочую область, решение продолжит работать без изменений.
> - Корпорация Майкрософт продолжит поддерживать отправку журналов ресурсов для групп безопасности сети в рабочую область с использованием параметров диагностики.

Группы безопасности сети поддерживают следующие журналы:

* NetworkSecurityGroupEvent
* NetworkSecurityGroupRuleCounter

### <a name="install-and-configure-the-solution"></a>Установка и настройка решения
Установите и настройте решение для анализа сетей Azure, выполнив следующие указания:

1. Включите решение "Аналитика групп безопасности сетей Azure", как описано в статье [Добавление решений для мониторинга в Azure Monitor из Коллекции решений](./solutions.md).
2. Включите ведение журнала диагностики для ресурсов [групп безопасности сети](../../virtual-network/virtual-network-nsg-manage-log.md), для которых требуется выполнять мониторинг.

### <a name="enable-azure-network-security-group-diagnostics-in-the-portal"></a>Включение диагностики групп безопасности сети Azure на портале

1. На портале Azure перейдите к ресурсу группы безопасности сети, который необходимо отслеживать.
2. Выберите *Журналы диагностики*, чтобы открыть следующую страницу:

   ![Снимок экрана: страница "журналы диагностики" для ресурса группы безопасности сети, в которой отображается параметр включения диагностики.](media/azure-networking-analytics/log-analytics-nsg-enable-diagnostics01.png)
3. Щелкните *Включить диагностику*, чтобы открыть следующую страницу:

   ![Снимок экрана страницы для настройки параметров диагностики. Параметр Status имеет значение ON, выбран параметр Send to Log Analytics и выбраны два типа журналов.](media/azure-networking-analytics/log-analytics-nsg-enable-diagnostics02.png)
4. Чтобы включить диагностику, нажмите кнопку *Вкл.* в разделе *Состояние*.
5. Установите флажок *Send to Log Analytics* (Отправить в Log Analytics).
6. Выберите существующую рабочую область Log Analytics или создайте новую.
7. В разделе **Журнал** установите флажки для всех типов журналов, которые необходимо собирать.
8. Щелкните *Сохранить*, чтобы включить ведение журнала диагностики в Log Analytics.

### <a name="enable-azure-network-diagnostics-using-powershell"></a>Включение диагностики сети Azure с помощью PowerShell

В следующем примере скрипта PowerShell показано, как включить ведение журналов ресурсов для групп безопасности сети.
```powershell
$workspaceId = "/subscriptions/d2e37fee-1234-40b2-5678-0b2199de3b50/resourcegroups/oi-default-east-us/providers/microsoft.operationalinsights/workspaces/rollingbaskets"

$nsg = Get-AzNetworkSecurityGroup -Name 'ContosoNSG'

Set-AzDiagnosticSetting -ResourceId $nsg.ResourceId  -WorkspaceId $workspaceId -Enabled $true
```

### <a name="use-azure-network-security-group-analytics"></a>Использование анализа групп безопасности сети Azure
Выбрав элемент **Azure Network Security Group analytics** (Анализ групп безопасности сети Azure) в разделе "Обзор", можно просмотреть сводные данные журналов и подробные сведения по следующим категориям:

* Заблокированные потоки группы безопасности сети:
  * правила группы безопасности сети с заблокированными потоками;
  * MAC-адреса c заблокированными потоками.
* Разрешенные потоки группы безопасности сети:
  * правила группы безопасности сети с разрешенными потоками;
  * MAC-адреса c разрешенными потоками.

![Снимок экрана с данными для заблокированных потоков группы безопасности сети, включая правила с заблокированными потоками и MAC-адресами с заблокированными потоками.](media/azure-networking-analytics/log-analytics-nsg01.png)

![Снимок экрана с данными для разрешенных потоков группы безопасности сети, включая правила с разрешенными потоками и MAC-адресами с разрешенными последовательностями.](media/azure-networking-analytics/log-analytics-nsg02.png)

На панели мониторинга **Azure Network Security Group analytics** (Анализ групп безопасности сети Azure) просмотрите сводные данные в колонках, а затем щелкните одну из них, чтобы просмотреть подробные сведения на странице поиска журналов.

На любой из страниц поиска журналов можно просмотреть результаты по времени, подробные результаты и историю поиска журналов. Для сужения области результатов выполните фильтрацию по аспектам.

## <a name="migrating-from-the-old-networking-analytics-solution"></a>Миграция из устаревшего решения для анализа сетевой активности
В январе 2017 г. был изменен поддерживаемый способ отправки журналов из Шлюзов приложений Azure и групп безопасности сети Azure в рабочую область Log Analytics. Эти изменения обеспечивают следующие преимущества:
+ Журналы записываются непосредственно в Azure Monitor без необходимости использовать учетную запись хранения.
+ Меньше задержка между моментом создания журналов и их доступностью в Azure Monitor.
+ Меньше этапов настройки.
+ Общий формат для всех типов системы диагностики Azure.

Чтобы использовать обновленные решения, выполните следующие действия:

1. [Настройте отправку данных диагностики из Шлюзов приложений Azure напрямую в Azure Monitor.](#enable-azure-application-gateway-diagnostics-in-the-portal)
2. [Настройте отправку данных диагностики из групп безопасности сети Azure напрямую в Azure Monitor.](#enable-azure-network-security-group-diagnostics-in-the-portal)
2. Включите решения *Аналитика службы приложений Azure* и *Аналитика групп безопасности сети Azure*, как описано в статье [Добавление решений для мониторинга в Azure Monitor из Коллекции решений](solutions.md).
3. Обновите все сохраненные запросы, панели мониторинга и оповещения, чтобы использовать новый тип данных.
   + Тип меняется на AzureDiagnostics. Параметр ResourceType можно использовать для фильтрации по журналам сети.

     | Вместо: | Используйте следующую команду: |
     | --- | --- |
     | NetworkApplicationgateways &#124; where OperationName=="ApplicationGatewayAccess" | AzureDiagnostics &#124; where ResourceType=="APPLICATIONGATEWAYS" and OperationName=="ApplicationGatewayAccess" |
     | NetworkApplicationgateways &#124; where OperationName=="ApplicationGatewayPerformance" | AzureDiagnostics &#124; where ResourceType=="APPLICATIONGATEWAYS" and OperationName=="ApplicationGatewayPerformance" |
     | NetworkSecuritygroups | AzureDiagnostics &#124; where ResourceType=="NETWORKSECURITYGROUPS" |

   + Для любого поля, имя которого содержит суффикс \_s, \_d или \_g, переведите первый знак в нижний регистр.
   + Для любого поля, имя которого содержит суффикс \_o, данные разбиваются на отдельные поля на основе имен вложенных полей.
4. Удалите устаревшее решение *Анализ сетевой активности Azure (не рекомендуется)* .
   + Если используется PowerShell, то выполните следующую команду: `Set-AzureOperationalInsightsIntelligencePack -ResourceGroupName <resource group that the workspace is in> -WorkspaceName <name of the log analytics workspace> -IntelligencePackName "AzureNetwork" -Enabled $false`

Данные, собранные до этого изменения, не отображаются в новом решении. Эти данные по-прежнему можно запрашивать с помощью старых имен типов и полей.

## <a name="troubleshooting"></a>Устранение неполадок
[!INCLUDE [log-analytics-troubleshoot-azure-diagnostics](../../../includes/log-analytics-troubleshoot-azure-diagnostics.md)]

## <a name="next-steps"></a>Дальнейшие действия
* Дополнительные сведения об обновлении данных см. в статье [Общие сведения о запросах журналов в Azure Monitor](../logs/log-query-overview.md).

