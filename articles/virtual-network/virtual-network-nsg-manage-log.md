---
title: Ведение журнала диагностики ресурсов для группы безопасности сети
titlesuffix: Azure Virtual Network
description: Узнайте, как включить журналы диагностических ресурсов событий и правил для группы безопасности сети Azure.
services: virtual-network
author: KumudD
manager: mtillman
ms.service: virtual-network
ms.topic: how-to
ms.workload: infrastructure-services
ms.date: 06/04/2018
ms.author: kumud
ms.openlocfilehash: bb078b9738e995a1c507f7934a7dd64f075d5fe0
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100596541"
---
# <a name="resource-logging-for-a-network-security-group"></a>Ведение журнала ресурсов для группы безопасности сети

Группа безопасности сети (NSG) содержит правила, которые разрешают или запрещают трафик для подсети виртуальной сети, сетевого интерфейса или и того, и другого. 

При включении ведения журнала для NSG можно собрать следующие типы данных журнала ресурсов:

* **Журналы событий**. Содержат записи, по которым правила NSG на основе MAC-адреса применяются к виртуальным машинам.
* **Журналы счетчиков**. Содержат записи с информацией о том, сколько раз каждое правило NSG было применено для запрета или разрешения трафика. Состояние этих правил собираются каждые 300 секунд.

Журналы ресурсов доступны только для группы безопасности сети, развернутых с помощью модели развертывания Azure Resource Manager. Вы не можете включить ведение журнала ресурсов для группы безопасности сети, развернутых с помощью классической модели развертывания. Чтобы получить более полное представление об этих двух моделях, прочитайте статью [Развертывание с помощью Azure Resource Manager и классическое развертывание: сведения о моделях развертывания и состоянии ресурсов](../azure-resource-manager/management/deployment-models.md?toc=%2fazure%2fvirtual-network%2ftoc.json).

Ведение журнала ресурсов включается отдельно для *каждого* NSG, для которого требуется сбор диагностических данных. Если вы заинтересованы в журналах активности (операционных), см. статью [ведение журнала действий](../azure-monitor/essentials/platform-logs-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json)Azure.

## <a name="enable-logging"></a>Включение ведения журналов

Для включения ведения журнала ресурсов можно использовать [портал Azure](#azure-portal), [PowerShell](#powershell)или [Azure CLI](#azure-cli) .

### <a name="azure-portal"></a>Портал Azure

1. Войдите на [портал](https://portal.azure.com).
2. Щелкните **Все службы**, затем введите *группы безопасности сети*. Когда элемент **Группы безопасности сети** появится в результатах поиска, выберите его.
3. Выберите NSG, для которой хотите включить ведение журнала.
4. В разделе **Мониторинг** выберите **Журналы диагностики** и выберите **Включить диагностику**, как показано на следующем рисунке.

   ![Включение диагностики](./media/virtual-network-nsg-manage-log/turn-on-diagnostics.png)

5. В разделе **Параметры диагностики** выберите или введите приведенные ниже сведения, а затем нажмите кнопку **Сохранить**.

    | Параметр                                                                                     | Значение                                                          |
    | ---------                                                                                   |---------                                                       |
    | Имя                                                                                        | Имя на ваш выбор.  Например: *myNsgDiagnostics*.      |
    | **Архивировать в учетной записи хранения**, **Передать в концентратор событий** и **Отправить в Log Analytics** | Можно выбрать любые назначения. Узнайте больше в разделе [Целевое расположение для журналов](#log-destinations).                                                                                                                                           |
    | LOG                                                                                         | Выберите одну или обе категории журналов. Чтобы узнать больше о данных, записываемых в каждую категорию, ознакомьтесь с разделом [Категории журналов](#log-categories).                                                                                                                                             |
6. Просмотрите и проанализируйте журналы. Дополнительные сведения см. в разделе [Просмотр и анализ журналов](#view-and-analyze-logs).

### <a name="powershell"></a>PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Вы можете выполнить приведенные ниже команды в [Azure Cloud Shell](https://shell.azure.com/powershell) или с помощью PowerShell на своем компьютере. Azure Cloud Shell — это бесплатная интерактивная оболочка. Она включает предварительно установленные общие инструменты Azure и настроена для использования с вашей учетной записью. При запуске PowerShell с компьютера необходим модуль Azure PowerShell версии 1.0.0 или более поздней. Выполните `Get-Module -ListAvailable Az` на компьютере, чтобы получить сведения об установленной версии. Если вам необходимо выполнить обновление, ознакомьтесь со статьей, посвященной [установке модуля Azure PowerShell](/powershell/azure/install-az-ps). Если вы используете PowerShell локально, вам также потребуется выполнить вход в `Connect-AzAccount` Azure с учетной записью, имеющей [необходимые разрешения](virtual-network-network-interface.md#permissions).

Чтобы включить ведение журнала ресурсов, необходим идентификатор существующего NSG. Если у вас нет существующего NSG, вы можете создать его с помощью [New-азнетворксекуритиграуп](/powershell/module/az.network/new-aznetworksecuritygroup).

Получите группу безопасности сети, для которой необходимо включить ведение журнала ресурсов, с помощью [Get-азнетворксекуритиграуп](/powershell/module/az.network/get-aznetworksecuritygroup). Например, чтобы получить NSG *myNsg* в группе ресурсов *myResourceGroup*, введите следующую команду.

```azurepowershell-interactive
$Nsg=Get-AzNetworkSecurityGroup `
  -Name myNsg `
  -ResourceGroupName myResourceGroup
```

Журналы ресурсов можно записывать в три типа назначения. Дополнительные сведения см. в разделе [Целевое расположение для журналов](#log-destinations). В этой статье журналы отправляются в *Log Analytics* в качестве примера. Получите существующую рабочую область Log Analytics с помощью [Get-азоператионалинсигхтсворкспаце](/powershell/module/az.operationalinsights/get-azoperationalinsightsworkspace). Например, чтобы получить рабочую область *myWorkspace* в группе ресурсов *myWorkspaces*, введите следующую команду:

```azurepowershell-interactive
$Oms=Get-AzOperationalInsightsWorkspace `
  -ResourceGroupName myWorkspaces `
  -Name myWorkspace
```

Если у вас нет рабочей области, вы можете создать ее с помощью [New-азоператионалинсигхтсворкспаце](/powershell/module/az.operationalinsights/new-azoperationalinsightsworkspace).

Существуют две категории, для которых можно включить журналы. Дополнительные сведения см. в разделе [Категории журналов](#log-categories). Включите ведение журнала ресурсов для NSG с помощью [Set-аздиагностиксеттинг](/powershell/module/az.monitor/set-azdiagnosticsetting). Следующий пример записывает в журнал данные о категориях событий и счетчиков и сохраняет его в рабочую область для NSG, используя идентификаторы NSG и рабочей области, полученные ранее.

```azurepowershell-interactive
Set-AzDiagnosticSetting `
  -ResourceId $Nsg.Id `
  -WorkspaceId $Oms.ResourceId `
  -Enabled $true
```

Если требуется записывать данные только одной из категорий, добавьте в предыдущую команду параметр `-Categories` и *NetworkSecurityGroupEvent* или *NetworkSecurityGroupRuleCounter*. Если вы хотите записывать журналы не в рабочую область Log Analytics, а в другое [назначение](#log-destinations), используйте соответствующие параметры [учетной записи хранения](../azure-monitor/essentials/resource-logs.md?toc=%2fazure%2fvirtual-network%2ftoc.json#send-to-azure-storage) или [концентратора событий](../azure-monitor/essentials/resource-logs.md?toc=%2fazure%2fvirtual-network%2ftoc.json#send-to-azure-event-hubs) Azure.

Просмотрите и проанализируйте журналы. Дополнительные сведения см. в разделе [Просмотр и анализ журналов](#view-and-analyze-logs).

### <a name="azure-cli"></a>Azure CLI

Вы можете выполнить приведенные ниже команды в [Azure Cloud Shell](https://shell.azure.com/bash) или Azure CLI на своем компьютере. Azure Cloud Shell — это бесплатная интерактивная оболочка. Она включает предварительно установленные общие инструменты Azure и настроена для использования с вашей учетной записью. Для запуска CLI на компьютере требуется версия 2.0.38 или выше. Выполните `az --version` на компьютере, чтобы получить сведения об установленной версии. Если вам необходимо выполнить обновление, см. статью [Установка Azure CLI](/cli/azure/install-azure-cli). Если CLI работает локально, необходимо также выполнить `az login`, чтобы войти в Azure с учетной записью, предоставляющей [необходимые разрешения](virtual-network-network-interface.md#permissions).

Чтобы включить ведение журнала ресурсов, необходим идентификатор существующего NSG. Если у вас нет NSG, ее можно создать, выполнив команду [az network nsg create](/cli/azure/network/nsg#az-network-nsg-create).

Получите группу безопасности сети, для которой необходимо включить ведение журнала ресурсов, с помощью команды [AZ Network NSG показывать](/cli/azure/network/nsg#az-network-nsg-show). Например, чтобы получить NSG *myNsg* в группе ресурсов *myResourceGroup*, введите следующую команду.

```azurecli-interactive
nsgId=$(az network nsg show \
  --name myNsg \
  --resource-group myResourceGroup \
  --query id \
  --output tsv)
```

Журналы ресурсов можно записывать в три типа назначения. Дополнительные сведения см. в разделе [Целевое расположение для журналов](#log-destinations). В этой статье журналы отправляются в *Log Analytics* в качестве примера. Дополнительные сведения см. в разделе [Категории журналов](#log-categories).

Включите ведение журнала ресурсов для NSG с помощью команды [AZ Monitor диагностики-Settings Create](/cli/azure/monitor/diagnostic-settings#az-monitor-diagnostic-settings-create). Следующий пример записывает данные событий и категории счетчика в существующую рабочую область с именем *myWorkspace*, которая существует в группе ресурсов с именем *myWorkspaces*, а также идентификатор группы безопасности сети, который вы получили ранее:

```azurecli-interactive
az monitor diagnostic-settings create \
  --name myNsgDiagnostics \
  --resource $nsgId \
  --logs '[ { "category": "NetworkSecurityGroupEvent", "enabled": true, "retentionPolicy": { "days": 30, "enabled": true } }, { "category": "NetworkSecurityGroupRuleCounter", "enabled": true, "retentionPolicy": { "days": 30, "enabled": true } } ]' \
  --workspace myWorkspace \
  --resource-group myWorkspaces
```

Если у вас нет существующей рабочей области, ее можно создать с помощью [портала Azure](../azure-monitor/logs/quick-create-workspace.md?toc=%2fazure%2fvirtual-network%2ftoc.json) или [PowerShell](/powershell/module/az.operationalinsights/new-azoperationalinsightsworkspace). Существуют две категории, для которых можно включить журналы.

Если вы хотите регистрировать данные только для какой-то из категорий, удалите ненужную категорию с помощью предыдущей команды. Если вы хотите записывать журналы не в рабочую область Log Analytics, а в другое [назначение](#log-destinations), используйте соответствующие параметры [учетной записи хранения](../azure-monitor/essentials/resource-logs.md?toc=%2fazure%2fvirtual-network%2ftoc.json#send-to-azure-storage) или [концентратора событий](../azure-monitor/essentials/resource-logs.md?toc=%2fazure%2fvirtual-network%2ftoc.json#send-to-azure-event-hubs) Azure.

Просмотрите и проанализируйте журналы. Дополнительные сведения см. в разделе [Просмотр и анализ журналов](#view-and-analyze-logs).

## <a name="log-destinations"></a>Целевое расположение для журналов

Назначения данных диагностики:
- [Запись в учетную запись хранения](../azure-monitor/essentials/resource-logs.md?toc=%2fazure%2fvirtual-network%2ftoc.json#send-to-azure-storage) для аудита или проверки вручную. В параметрах диагностики ресурсов можно задать время хранения (в днях).
- [Потоковая передача журнала в концентратор событий](../azure-monitor/essentials/resource-logs.md?toc=%2fazure%2fvirtual-network%2ftoc.json#send-to-azure-event-hubs) для приема сторонней службой или пользовательским аналитическим решением, например PowerBI.
- [Записывается в журналы Azure Monitor](../azure-monitor/essentials/resource-logs.md?toc=%2fazure%2fvirtual-network%2ftoc.json#send-to-azure-storage).

## <a name="log-categories"></a>Категории журналов

Данные в формате JSON записываются в журналы следующих категорий.

### <a name="event"></a>Событие

Журнал событий содержит сведения о том, какие правила NSG на основе MAC-адреса применяются к виртуальным машинам. Эти данные регистрируются для каждого события. В следующем примере данные зарегистрированы для виртуальной машины с IP-адресом 192.168.1.4 и MAC-адресом 00-0D-3A-92-6A-7C.

```json
{
    "time": "[DATE-TIME]",
    "systemId": "[ID]",
    "category": "NetworkSecurityGroupEvent",
    "resourceId": "/SUBSCRIPTIONS/[SUBSCRIPTION-ID]/RESOURCEGROUPS/[RESOURCE-GROUP-NAME]/PROVIDERS/MICROSOFT.NETWORK/NETWORKSECURITYGROUPS/[NSG-NAME]",
    "operationName": "NetworkSecurityGroupEvents",
    "properties": {
        "vnetResourceGuid":"[ID]",
        "subnetPrefix":"192.168.1.0/24",
        "macAddress":"00-0D-3A-92-6A-7C",
        "primaryIPv4Address":"192.168.1.4",
        "ruleName":"[SECURITY-RULE-NAME]",
        "direction":"[DIRECTION-SPECIFIED-IN-RULE]",
        "priority":"[PRIORITY-SPECIFIED-IN-RULE]",
        "type":"[ALLOW-OR-DENY-AS-SPECIFIED-IN-RULE]",
        "conditions":{
            "protocols":"[PROTOCOLS-SPECIFIED-IN-RULE]",
            "destinationPortRange":"[PORT-RANGE-SPECIFIED-IN-RULE]",
            "sourcePortRange":"[PORT-RANGE-SPECIFIED-IN-RULE]",
            "sourceIP":"[SOURCE-IP-OR-RANGE-SPECIFIED-IN-RULE]",
            "destinationIP":"[DESTINATION-IP-OR-RANGE-SPECIFIED-IN-RULE]"
            }
        }
}
```

### <a name="rule-counter"></a>Счетчик правил

Журнал счетчика правил содержит сведения о всех случаях применения правил к ресурсам. Данные, представленные в этом примере, регистрируются для каждого применения правила. В следующем примере данные зарегистрированы для виртуальной машины с IP-адресом 192.168.1.4 и MAC-адресом 00-0D-3A-92-6A-7C.

```json
{
    "time": "[DATE-TIME]",
    "systemId": "[ID]",
    "category": "NetworkSecurityGroupRuleCounter",
    "resourceId": "/SUBSCRIPTIONS/[SUBSCRIPTION ID]/RESOURCEGROUPS/[RESOURCE-GROUP-NAME]/PROVIDERS/MICROSOFT.NETWORK/NETWORKSECURITYGROUPS/[NSG-NAME]",
    "operationName": "NetworkSecurityGroupCounters",
    "properties": {
        "vnetResourceGuid":"[ID]",
        "subnetPrefix":"192.168.1.0/24",
        "macAddress":"00-0D-3A-92-6A-7C",
        "primaryIPv4Address":"192.168.1.4",
        "ruleName":"[SECURITY-RULE-NAME]",
        "direction":"[DIRECTION-SPECIFIED-IN-RULE]",
        "type":"[ALLOW-OR-DENY-AS-SPECIFIED-IN-RULE]",
        "matchedConnections":125
        }
}
```

> [!NOTE]
> Исходный IP-адрес для обмена данными записывается. Однако можно включить [ведение журнала потока NSG](../network-watcher/network-watcher-nsg-flow-logging-portal.md) для NSG. В этом случае в журнал будут записываться все сведения о счетчике правил, а также исходный IP-адрес, с которого был инициирован обмен данными. Данные журнала потоков NSG записываются в учетную запись хранения Azure. Данные можно анализировать с помощью функции [анализа трафика](../network-watcher/traffic-analytics.md) службы "Наблюдатель за сетями".

## <a name="view-and-analyze-logs"></a>Просмотр и анализ журналов

Сведения о просмотре данных журнала ресурсов см. в статье [Общие сведения о журналах платформы Azure](../azure-monitor/essentials/platform-logs-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json). Если данные диагностики отправляются в:
- **Журналы Azure Monitor**. Вы можете использовать решение для [анализа групп безопасности сети](../azure-monitor/insights/azure-networking-analytics.md?toc=%2fazure%2fvirtual-network%2ftoc.json#azure-network-security-group-analytics-solution-in-azure-monitor
) для получения расширенных сведений. Это решение наглядно представляет правила NSG, которые разрешают или запрещают трафик по MAC-адресу сетевого интерфейса в виртуальной машине.
- **учетную запись хранения**, то данные записываются в файл PT1H.json. Расположение журналов:
  - журнал событий: `insights-logs-networksecuritygroupevent/resourceId=/SUBSCRIPTIONS/[ID]/RESOURCEGROUPS/[RESOURCE-GROUP-NAME-FOR-NSG]/PROVIDERS/MICROSOFT.NETWORK/NETWORKSECURITYGROUPS/[NSG NAME]/y=[YEAR]/m=[MONTH/d=[DAY]/h=[HOUR]/m=[MINUTE]`
  - журнал счетчика правил: `insights-logs-networksecuritygrouprulecounter/resourceId=/SUBSCRIPTIONS/[ID]/RESOURCEGROUPS/[RESOURCE-GROUP-NAME-FOR-NSG]/PROVIDERS/MICROSOFT.NETWORK/NETWORKSECURITYGROUPS/[NSG NAME]/y=[YEAR]/m=[MONTH/d=[DAY]/h=[HOUR]/m=[MINUTE]`

## <a name="next-steps"></a>Дальнейшие действия

- Дополнительные сведения о [ведении журнала действий](../azure-monitor/essentials/platform-logs-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json). Ведение журнала действий включено по умолчанию для всех создаваемых NSG, независимо от модели развертывания Azure. Чтобы определить в журнале активности, какие операции были выполнены с группами безопасности сети, найдите записи, содержащие следующие типы ресурсов:
  - Microsoft.ClassicNetwork/networkSecurityGroups
  - Microsoft.ClassicNetwork/networkSecurityGroups/securityRules
  - Microsoft.Network/networkSecurityGroups
  - Microsoft.Network/networkSecurityGroups/securityRules
- Чтобы узнать, как вести журнал диагностики и записывать в него исходные IP-адреса для каждого потока, ознакомьтесь с разделом [Руководство по регистрации потока сетевого трафика на виртуальную машину и с нее с помощью портала Azure](../network-watcher/network-watcher-nsg-flow-logging-portal.md?toc=%2fazure%2fvirtual-network%2ftoc.json).
