---
title: Мониторинг журналов и метрик Брандмауэра Azure
description: Из этой статьи вы узнаете, как включить журналы и метрики брандмауэра Azure и управлять ими.
services: firewall
author: vhorne
ms.service: firewall
ms.topic: how-to
ms.date: 11/04/2020
ms.author: victorh
ms.openlocfilehash: 52c6ef9edfc42bf1ad3b3279e0fa4e19b4cf502c
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98788270"
---
# <a name="monitor-azure-firewall-logs-and-metrics"></a>Мониторинг журналов и метрик Брандмауэра Azure

Работу брандмауэра Azure можно отслеживать с помощью журналов брандмауэра. Также журналы действий можно использовать для аудита операций на ресурсах брандмауэра Azure. С помощью метрик можно просматривать счетчики производительности на портале.

Доступ к некоторым из этих журналов можно получить через портал. Журналы можно отправлять в [Azure Monitor журналы](../azure-monitor/insights/azure-networking-analytics.md), хранилище и концентраторы событий, а также анализировать в журналах Azure Monitor или с помощью различных средств, таких как Excel и Power BI.

[!INCLUDE [azure-monitor-log-analytics-rebrand](../../includes/azure-monitor-log-analytics-rebrand.md)]

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Предварительные условия

Прежде чем начать, ознакомьтесь с [журналами и метриками брандмауэра Azure](logs-and-metrics.md) , чтобы получить общие сведения о журналах диагностики и метриках, доступных для брандмауэра Azure.

## <a name="enable-diagnostic-logging-through-the-azure-portal"></a>Включение журнала ведения диагностики на портале Azure

Для включения ведения журнала диагностики потребуется несколько минут, чтобы данные появились в журналах после завершения этой процедуры. Если ничего не происходит, проверьте еще раз через несколько минут.

1. В портал Azure откройте группу ресурсов брандмауэра и выберите брандмауэр.
2. В разделе **Мониторинг** выберите **Параметры диагностики**.

   Для брандмауэра Azure доступны четыре журнала, относящиеся к службе:

   * AzureFirewallApplicationRule
   * AzureFirewallNetworkRule
   * азурефиреваллсреатинтеллог
   * азурефиреваллднспрокси


3. Выберите **Добавить параметр диагностики**. На странице **Параметры диагностики** представлены параметры журналов диагностики.
5. В этом примере журналы хранятся в Журналах Azure Monitor, поэтому введите имя **Firewall log analytics**.
6. В разделе **Журнал** выберите **азурефиреваллаппликатионруле**, **азурефиреваллнетворкруле**, **азурефиреваллсреатинтеллог** и **AzureFirewallDnsProxy** , чтобы получить журналы.
7. Выберите **отправить log Analytics** , чтобы настроить рабочую область.
8. Выберите свою подписку.
9. Щелкните **Сохранить**.

## <a name="enable-diagnostic-logging-by-using-powershell"></a>Включение ведения журнала диагностики с помощью PowerShell

Ведение журнала действий автоматически включается для каждого ресурса Resource Manager. Чтобы начать сбор соответствующих данных, журналы ведения диагностики должны быть включены.

Чтобы включить ведение журнала диагностики с помощью PowerShell, выполните следующие действия.

1. Запишите идентификатор ресурса рабочей области Log Analytics, где хранятся данные журнала. Это значение имеет вид: `/subscriptions/<subscriptionId>/resourceGroups/<resource group name>/providers/microsoft.operationalinsights/workspaces/<workspace name>` .

   Вы можете использовать любую рабочую область в подписке. Получить эти сведения можно на портале Azure. Эти сведения находятся на странице **свойств** ресурсов.

2. Запишите или запомните идентификатор ресурса брандмауэра, для которого нужно включить ведение журнала, Это значение имеет вид: `/subscriptions/<subscriptionId>/resourceGroups/<resource group name>/providers/Microsoft.Network/azureFirewalls/<Firewall name>` .

   Получить эти сведения можно на портале.

3. Включите ведение журнала диагностики для всех журналов и метрик с помощью следующего командлета PowerShell:

   ```powershell
   $diagSettings = @{
      Name = 'toLogAnalytics'
      ResourceId = '/subscriptions/<subscriptionId>/resourceGroups/<resource group name>/providers/Microsoft.Network/azureFirewalls/<Firewall name>'
      WorkspaceId = '/subscriptions/<subscriptionId>/resourceGroups/<resource group name>/providers/microsoft.operationalinsights/workspaces/<workspace name>'
      Enabled = $true
   }
   Set-AzDiagnosticSetting  @diagSettings 
   ```

## <a name="enable-diagnostic-logging-by-using-the-azure-cli"></a>Включение ведения журнала диагностики с помощью Azure CLI

Ведение журнала действий автоматически включается для каждого ресурса Resource Manager. Чтобы начать сбор соответствующих данных, журналы ведения диагностики должны быть включены.

Чтобы включить ведение журнала диагностики с Azure CLI, выполните следующие действия.

1. Запишите идентификатор ресурса рабочей области Log Analytics, где хранятся данные журнала. Это значение имеет вид: `/subscriptions/<subscriptionId>/resourceGroups/<resource group name>/providers/Microsoft.Network/azureFirewalls/<Firewall name>` .

   Вы можете использовать любую рабочую область в подписке. Получить эти сведения можно на портале Azure. Эти сведения находятся на странице **свойств** ресурсов.

2. Запишите или запомните идентификатор ресурса брандмауэра, для которого нужно включить ведение журнала, Это значение имеет вид: `/subscriptions/<subscriptionId>/resourceGroups/<resource group name>/providers/Microsoft.Network/azureFirewalls/<Firewall name>` .

   Получить эти сведения можно на портале.

3. Включите ведение журнала диагностики для всех журналов и метрик с помощью следующей команды Azure CLI:

   ```azurecli-interactive
   az monitor diagnostic-settings create -n 'toLogAnalytics'
      --resource '/subscriptions/<subscriptionId>/resourceGroups/<resource group name>/providers/Microsoft.Network/azureFirewalls/<Firewall name>'
      --workspace '/subscriptions/<subscriptionId>/resourceGroups/<resource group name>/providers/microsoft.operationalinsights/workspaces/<workspace name>'
      --logs '[{\"category\":\"AzureFirewallApplicationRule\",\"Enabled\":true}, {\"category\":\"AzureFirewallNetworkRule\",\"Enabled\":true}, {\"category\":\"AzureFirewallDnsProxy\",\"Enabled\":true}]' 
      --metrics '[{\"category\": \"AllMetrics\",\"enabled\": true}]'
   ```

## <a name="view-and-analyze-the-activity-log"></a>Просмотр и анализ журнала действий

Данные журнала действий можно просматривать и анализировать с помощью любого из следующих методов:

* **Средства Azure.** Информацию из журналов действий можно получать с помощью Azure PowerShell, Azure CLI, REST API Azure или портала Azure. Пошаговые инструкции для каждого метода подробно описаны в статье [Activity operations with Resource Manager](../azure-resource-manager/management/view-activity-logs.md) (Выполнение операций в журналах действий с помощью Resource Manager).
* **Power BI**. Если у вас еще нет учетной записи [Power BI](https://powerbi.microsoft.com/pricing) , вы можете попробовать ее бесплатно. Используя [пакет содержимого журналов действий Azure для Power BI](https://powerbi.microsoft.com/en-us/documentation/powerbi-content-pack-azure-audit-logs/), можно анализировать данные с помощью предварительно настроенных панелей мониторинга, которые можно использовать "как есть" или дополнительно настроить.
* **Azure Sentinel**. Вы можете подключать журналы брандмауэра Azure к Azure Sentinel, позволяя просматривать данные журналов в книгах, использовать их для создания пользовательских оповещений и внедрять их для улучшения вашего расследования. Соединитель данных брандмауэра Azure в Azure Sentinel сейчас находится в общедоступной предварительной версии. Дополнительные сведения см. в статье [подключение данных из брандмауэра Azure](../sentinel/connect-azure-firewall.md).

## <a name="view-and-analyze-the-network-and-application-rule-logs"></a>Просмотр и анализ журналов правил сети и приложений

Служба [Журналы Azure Monitor](../azure-monitor/insights/azure-networking-analytics.md) собирает файлы счетчика и журнала событий. Эта служба предоставляет средства визуализации и эффективные возможности поиска для анализа журналов.

Примеры запросов к службе анализа журналов в Azure Firewall [см. в](./firewall-workbook.md)этой статье.

[Книга брандмауэра Azure](firewall-workbook.md) предоставляет гибкий холст для анализа данных брандмауэра Azure. Его можно использовать для создания многофункциональных визуальных отчетов в портал Azure. Вы можете коснуться нескольких брандмауэров, развернутых в Azure, и объединить их в единый интерактивный интерфейс.

Вы также можете подключиться к учетной записи хранения и извлечь записи журнала JSON для журналов доступа и производительности. После скачивания JSON-файлов их можно преобразовать в формат CSV и просматривать в Excel, Power BI или другом средстве визуализации данных.

> [!TIP]
> Если вы знакомы с Visual Studio и основными понятиями изменения значений констант и переменных в C#, можно использовать [инструменты преобразования журналов](https://github.com/Azure-Samples/networking-dotnet-log-converter), доступные на сайте GitHub.

## <a name="view-metrics"></a>Просмотр метрик
Перейдите к брандмауэру Azure, в разделе **мониторинг** выберите **метрики**. Чтобы просмотреть доступные значения, выберите раскрывающийся список **Метрика**.

## <a name="next-steps"></a>Дальнейшие действия

Теперь, когда брандмауэр настроен для сбора журналов, можно открыть Журналы Azure Monitor, чтобы просмотреть данные.

[Мониторинг журналов с помощью книги брандмауэра Azure](firewall-workbook.md)

[Решения для мониторинга сетей в Журналах Azure Monitor](../azure-monitor/insights/azure-networking-analytics.md)