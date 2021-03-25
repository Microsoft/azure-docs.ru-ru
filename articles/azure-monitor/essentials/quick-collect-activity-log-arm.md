---
title: Отправка журнала действий Azure в рабочую область Log Analytics с помощью шаблона Azure Resource Manager
description: Используйте шаблоны ARM, чтобы создать параметр диагностики и рабочую область Log Analytics для отправки журнала действий в журналы Azure Monitor.
ms.topic: quickstart
ms.custom: subject-armqs, devx-track-azurecli
author: bwren
ms.author: bwren
ms.date: 06/25/2020
ms.openlocfilehash: 56810cffcb9665810c452276be34e6924fd992b2
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102033288"
---
# <a name="quickstart-send-azure-activity-log-to-log-analytics-workspace-using-an-arm-template"></a>Краткое руководство. Отправка журнала действий Azure в рабочую область Log Analytics с помощью шаблона ARM

Журнал действий — это журнал платформы в Azure, который предоставляет аналитические сведения о событиях уровня подписки, например об изменении ресурса или запуске виртуальной машины. Вы можете просмотреть журнал действий на портале Azure или получить записи с помощью PowerShell и CLI. В этом кратком руководстве показано, как использовать шаблоны Azure Resource Manager (шаблоны ARM) для создания параметра диагностики и рабочей области Log Analytics и отправки журнала действий в журналы Azure Monitor. Там их можно анализировать с помощью [запросов журнала](../logs/log-query-overview.md) и включать другие возможности, такие как [оповещения журнала](../alerts/alerts-log-query.md) и [книги](../visualize/workbooks-overview.md).

[!INCLUDE [About Azure Resource Manager](../../../includes/resource-manager-quickstart-introduction.md)]

## <a name="prerequisites"></a>Предварительные требования

- Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.
- Чтобы выполнять команды с локального компьютера, установите Azure CLI или модули Azure PowerShell. Дополнительные сведения см. в статьях [Установка Azure CLI](/cli/azure/install-azure-cli) и [Установка Azure PowerShell](/powershell/azure/install-az-ps).

## <a name="create-a-log-analytics-workspace"></a>Создание рабочей области Log Analytics

### <a name="review-the-template"></a>Изучение шаблона

Следующий шаблон создает пустую рабочую область Log Analytics. Сохраните этот шаблон как файл *CreateWorkspace.json*.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "workspaceName": {
      "type": "string",
      "metadata": {
        "description": "Name of the workspace."
      }
    },
    "sku": {
      "type": "string",
      "defaultValue": "pergb2018",
      "allowedValues": [
        "pergb2018",
        "Free",
        "Standalone",
        "PerNode",
        "Standard",
        "Premium"
      ],
      "metadata": {
        "description": "Pricing tier: PerGB2018 or legacy tiers (Free, Standalone, PerNode, Standard or Premium) which are not available to all customers."
      }
    },
    "location": {
      "type": "string",
      "allowedValues": [
        "australiacentral",
        "australiaeast",
        "australiasoutheast",
        "brazilsouth",
        "canadacentral",
        "centralindia",
        "centralus",
        "eastasia",
        "eastus",
        "eastus2",
        "francecentral",
        "japaneast",
        "koreacentral",
        "northcentralus",
        "northeurope",
        "southafricanorth",
        "southcentralus",
        "southeastasia",
        "switzerlandnorth",
        "switzerlandwest",
        "uksouth",
        "ukwest",
        "westcentralus",
        "westeurope",
        "westus",
        "westus2"
      ],
      "metadata": {
        "description": "Specifies the location for the workspace."
      }
    },
    "retentionInDays": {
      "type": "int",
      "defaultValue": 120,
      "metadata": {
        "description": "Number of days to retain data."
      }
    },
    "resourcePermissions": {
      "type": "bool",
      "defaultValue": true,
      "metadata": {
        "description": "true to use resource or workspace permissions. false to require workspace permissions."
      }
    }
  },
  "resources": [
    {
      "type": "Microsoft.OperationalInsights/workspaces",
      "apiVersion": "2020-08-01",
      "name": "[parameters('workspaceName')]",
      "location": "[parameters('location')]",
      "properties": {
        "sku": {
          "name": "[parameters('sku')]"
        },
        "retentionInDays": "[parameters('retentionInDays')]",
        "features": {
          "searchVersion": 1,
          "legacy": 0,
          "enableLogAccessUsingOnlyResourcePermissions": "[parameters('resourcePermissions')]"
        }
      }
    }
  ]
}
```

Шаблон определяет один ресурс:

- [Microsoft.OperationalInsights/workspaces](/azure/templates/microsoft.operationalinsights/workspaces).

### <a name="deploy-the-template"></a>Развертывание шаблона

Разверните шаблон с помощью любого стандартного метода [развертывания шаблона ARM](../../azure-resource-manager/templates/deploy-portal.md), например с помощью CLI и PowerShell, как в примерах ниже. Замените примеры значений для **группы ресурсов**, **workspaceName** и **location** соответствующими значениями из вашей среды. Имя рабочей области должно быть уникальным в пределах всех подписок Azure.

# <a name="cli"></a>[CLI](#tab/CLI)

```azurecli
az login
az deployment group create \
    --name CreateWorkspace \
    --resource-group my-resource-group \
    --template-file CreateWorkspace.json \
    --parameters workspaceName='my-workspace-01' location='eastus'

```

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

```powershell
Connect-AzAccount
Select-AzSubscription -SubscriptionName my-subscription
New-AzResourceGroupDeployment -Name AzureMonitorDeployment -ResourceGroupName my-resource-group -TemplateFile CreateWorkspace.json -workspaceName my-workspace-01 -location eastus
```

---

### <a name="validate-the-deployment"></a>Проверка развертывания

Убедитесь, что рабочая область создана с помощью одной из следующих команд. Замените примеры значений для **группы ресурсов** и **workspaceName** значениями, которые вы использовали выше.

# <a name="cli"></a>[CLI](#tab/CLI)

```azurecli
az monitor log-analytics workspace show --resource-group my-workspace-01 --workspace-name my-resource-group
```

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

```powershell
Get-AzOperationalInsightsWorkspace -Name my-workspace-01 -ResourceGroupName my-resource-group
```

---

## <a name="create-diagnostic-setting"></a>Создание параметра диагностики

### <a name="review-the-template"></a>Изучение шаблона

Следующий шаблон создает параметр диагностики, который отправляет журнал действий в рабочую область Log Analytics. Сохраните этот шаблон как файл *CreateDiagnosticSetting.json*.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "settingName": {
        "type": "String"
    },
    "workspaceId": {
        "type": "String"
    }
  },
  "resources": [
    {
      "type": "Microsoft.Insights/diagnosticSettings",
      "apiVersion": "2017-05-01-preview",
      "name": "[parameters('settingName')]",
      "dependsOn": [],
      "properties": {
        "workspaceId": "[parameters('workspaceId')]",
        "logs": [
          {
          "category": "Administrative",
          "enabled": true
          },
          {
          "category": "Alert",
          "enabled": true
          },
          {
          "category": "Autoscale",
          "enabled": true
          },
          {
          "category": "Policy",
          "enabled": true
          },
          {
          "category": "Recommendation",
          "enabled": true
          },
          {
          "category": "ResourceHealth",
          "enabled": true
          },
          {
          "category": "Security",
          "enabled": true
          },
          {
          "category": "ServiceHealth",
          "enabled": true
          }
        ]
      }
    }
  ]
}
```

Шаблон определяет один ресурс:

- [Microsoft.Insights/diagnosticSettings](/azure/templates/microsoft.insights/diagnosticsettings).

### <a name="deploy-the-template"></a>Развертывание шаблона

Разверните шаблон с помощью любого стандартного метода [развертывания шаблона ARM](../../azure-resource-manager/templates/deploy-portal.md), например с помощью CLI и PowerShell, как в примерах ниже. Замените примеры значений для **группы ресурсов**, **workspaceName** и **location** соответствующими значениями из вашей среды. Имя рабочей области должно быть уникальным в пределах всех подписок Azure.

# <a name="cli"></a>[CLI](#tab/CLI)

```azurecli
az deployment sub create --name CreateDiagnosticSetting --location eastus --template-file CreateDiagnosticSetting.json --parameters settingName='Send Activity log to workspace' workspaceId='/subscriptions/00000000-0000-0000-0000-000000000000/resourcegroups/my-resource-group/providers/microsoft.operationalinsights/workspaces/my-workspace-01'

```

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

```powershell
New-AzSubscriptionDeployment -Name CreateDiagnosticSetting -location eastus -TemplateFile CreateDiagnosticSetting.json -settingName "Send Activity log to workspace" -workspaceId "/subscriptions/00000000-0000-0000-0000-000000000000/resourcegroups/my-resource-group/providers/microsoft.operationalinsights/workspaces/my-workspace-01"
```
---

### <a name="validate-the-deployment"></a>Проверка развертывания

С помощью одной из следующих команд проверьте, создан ли параметр диагностики. Замените примеры значений для подписки и имя параметра значениями, которые вы использовали выше.

> [!NOTE]
> Сейчас получить параметры диагностики на уровне подписки с помощью PowerShell нельзя.

```azurecli
az monitor diagnostic-settings show --resource '/subscriptions/00000000-0000-0000-0000-000000000000' --name 'Send Activity log to workspace'
```

## <a name="generate-log-data"></a>Создание данных журнала

В рабочую область Log Analytics будут отправляться только новые записи журнала действий, поэтому выполните в подписке действия, которые будут регистрироваться, например запустите или остановите виртуальную машину либо создайте или измените другой ресурс. Создание параметра диагностики и запись данных в рабочую область может занять несколько минут. После этого все события, записанные в журнал действий, будут отправлены в рабочую область в течение нескольких секунд.

## <a name="retrieve-data-with-a-log-query"></a>Получение данных путем отправки запроса к журналу

Используйте портал Azure, чтобы получить данные из рабочей области с помощью Log Analytics. На портале Azure найдите и выберите **Монитор**.

![Портал Azure](media/quick-collect-activity-log/azure-portal-monitor.png)

Выберите **Журналы** в меню **Azure Monitor**. Закройте страницу **Примеры запросов**. Если для созданной рабочей области не задана область, щелкните **Выбрать область** и найдите ее.

![Область Log Analytics](media/quick-collect-activity-log/log-analytics-scope.png)

В окне запроса введите `AzureActivity` и щелкните **Выполнить**. Этот простой запрос возвращает все записи в таблице *AzureActivity*, где хранятся все записи, отправленные из журнала действий.

![Простой запрос](media/quick-collect-activity-log/query-01.png)

Разверните одну из записей, чтобы просмотреть ее подробные свойства.

![Развертывание свойств](media/quick-collect-activity-log/expand-properties.png)

Попробуйте отправить более сложный запрос, например `AzureActivity | summarize count() by CategoryValue`, который возвращает количество событий, суммированных по категориям.

![Сложный запрос](media/quick-collect-activity-log/query-02.png)

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы планируете продолжить работу с последующими краткими руководствами и статьями, эти ресурсы можно не удалять. Удалите группу ресурсов, если она больше не нужна. Правила генерации оповещений и связанные ресурсы также будут удалены. Вы можете удалить группу ресурсов с помощью Azure CLI или Azure PowerShell.

# <a name="cli"></a>[CLI](#tab/CLI)

```azurecli
az group delete --name my-resource-group
```

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

```powershell
Remove-AzResourceGroup -Name my-resource-group
```

---

## <a name="next-steps"></a>Дальнейшие действия

Из этого краткого руководства вы узнали, как настроить отправку журнала действий в рабочую область Log Analytics. Теперь вы можете настроить сбор других данных в рабочую область, где их можно анализировать с помощью [запросов к журналам](../logs/log-query-overview.md) в Azure Monitor, и применять другие возможности, например [оповещения журналов](../alerts/alerts-log-query.md) и [книги](../visualize/workbooks-overview.md). Затем вам нужно собрать [журналы ресурсов](../essentials/resource-logs.md) из ресурсов Azure, которые дополняют данные в журнале действий. Так вы сможете получить представление об операциях, выполненных в пределах каждого ресурса.

> [!div class="nextstepaction"]
> [Получение и анализ журналов ресурсов с помощью Azure Monitor](../essentials/tutorial-resource-logs.md)