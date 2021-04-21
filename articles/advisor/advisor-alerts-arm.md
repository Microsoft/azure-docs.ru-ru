---
title: Создание оповещений Помощника по Azure для новых рекомендаций с помощью шаблона Resource Manager
description: Узнайте, как в Помощнике по Azure настроить оповещение для новых рекомендаций с помощью шаблона Azure Resource Manager (шаблона ARM).
ms.topic: quickstart
ms.custom: subject-armqs, devx-track-azurepowershell
ms.date: 06/29/2020
ms.openlocfilehash: 716ab104ce6517aeb554b42522e5906829877259
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107765673"
---
# <a name="quickstart-create-azure-advisor-alerts-on-new-recommendations-using-an-arm-template"></a>Краткое руководство. Создание оповещений Помощника по Azure для новых рекомендаций с помощью шаблона ARM

В этой статье описывается, как из Помощника по Azure настроить оповещение для новых рекомендаций с помощью шаблона Azure Resource Manager (шаблона ARM).

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Каждый раз, когда Помощник по Azure обнаруживает новую рекомендацию для одного из ваших ресурсов, событие сохраняется в [журнале действий Azure](../azure-monitor/essentials/platform-logs-overview.md). Вы можете настроить оповещения для этих событий из Помощника по Azure с помощью интерфейса создания оповещений для конкретных рекомендаций. Вы можете выбрать подписку и при необходимости группу ресурсов, чтобы определить ресурсы, по которым требуется получать оповещения.

Вы также можете определить типы рекомендаций с помощью следующих свойств.

- Категория
- Уровень влияния
- Тип рекомендации

Вы также можете настроить действие, которое будет выполняться при активации оповещения одним из следующих действий:

- выбор существующей группы действий;
- создание новой группы действий.

Дополнительные сведения о группах действий см. в разделе [Создание групп действий и управление ими на портале Azure](../azure-monitor/alerts/action-groups.md).

> [!NOTE]
> Сейчас оповещения Помощника доступны только для рекомендаций, связанных с высоким уровнем доступности, производительностью и расходами. Рекомендации, связанные с безопасностью, не поддерживаются.

## <a name="prerequisites"></a>Предварительные требования

- Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.
- Чтобы выполнять команды с локального компьютера, установите Azure CLI или модули Azure PowerShell. Дополнительные сведения см. в статьях [Установка Azure CLI](/cli/azure/install-azure-cli) и [Установка Azure PowerShell](/powershell/azure/install-az-ps).

## <a name="review-the-template"></a>Изучение шаблона

Ниже приведен шаблон, который создает группу действий с целевым объектом электронной почты и активирует все уведомления о работоспособности службы в целевой подписке. Сохраните этот шаблон как файл *CreateAdvisorAlert.json*.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "actionGroups_name": {
      "defaultValue": "advisorAlert",
      "type": "string"
    },
    "activityLogAlerts_name": {
      "defaultValue": "AdvisorAlertsTest",
      "type": "string"
    },
    "emailAddress": {
      "defaultValue": "<email address>",
      "type": "string"
    }
  },
  "variables": {
    "alertScope": "[concat('/','subscriptions','/',subscription().subscriptionId)]"
  },
  "resources": [
    {
      "comments": "Action Group",
      "type": "microsoft.insights/actionGroups",
      "apiVersion": "2019-06-01",
      "name": "[parameters('actionGroups_name')]",
      "location": "Global",
      "scale": null,
      "dependsOn": [],
      "tags": {},
      "properties": {
        "groupShortName": "[parameters('actionGroups_name')]",
        "enabled": true,
        "emailReceivers": [
          {
            "name": "[parameters('actionGroups_name')]",
            "emailAddress": "[parameters('emailAddress')]"
          }
        ],
        "smsReceivers": [],
        "webhookReceivers": []
      }
    },
    {
      "comments": "Azure Advisor Activity Log Alert",
      "type": "microsoft.insights/activityLogAlerts",
      "apiVersion": "2017-04-01",
      "name": "[parameters('activityLogAlerts_name')]",
      "location": "Global",
      "scale": null,
      "tags": {},
      "properties": {
        "scopes": [
          "[variables('alertScope')]"
        ],
        "condition": {
          "allOf": [
            {
              "field": "category",
              "equals": "Recommendation"
            },
            {
              "field": "properties.recommendationCategory",
              "equals": "Cost"
            },
            {
              "field": "properties.recommendationImpact",
              "equals": "Medium"
            },
            {
              "field": "operationName",
              "equals": "Microsoft.Advisor/recommendations/available/action"
            }
          ]
        },
        "actions": {
          "actionGroups": [
            {
              "actionGroupId": "[resourceId('microsoft.insights/actionGroups', parameters('actionGroups_name'))]",
              "webhookProperties": {}
            }
          ]
        },
        "enabled": true,
        "description": ""
      },
      "dependsOn": [
        "[resourceId('microsoft.insights/actionGroups', parameters('actionGroups_name'))]"
      ]
    }
  ]
}
```

Шаблон определяет два ресурса:

- [Microsoft.Insights/actionGroups](/azure/templates/microsoft.insights/actiongroups)
- [Microsoft.Insights/activityLogAlerts](/azure/templates/microsoft.insights/activityLogAlerts)

## <a name="deploy-the-template"></a>Развертывание шаблона

Разверните шаблон с помощью любого стандартного метода [развертывания шаблона ARM](../azure-resource-manager/templates/deploy-portal.md), например с помощью CLI и PowerShell, как в примерах ниже. Замените примеры значений для **группы ресурсов** и **emailAddress** соответствующими значениями для вашей среды. Имя рабочей области должно быть уникальным в пределах всех подписок Azure.

# <a name="cli"></a>[CLI](#tab/CLI)

```azurecli
az login
az deployment group create --name CreateAdvisorAlert --resource-group my-resource-group --template-file CreateAdvisorAlert.json --parameters emailAddress='user@contoso.com'
```

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

```powershell
Connect-AzAccount
Select-AzSubscription -SubscriptionName my-subscription
New-AzResourceGroupDeployment -Name CreateAdvisorAlert -ResourceGroupName my-resource-group -TemplateFile CreateAdvisorAlert.json -emailAddress user@contoso.com
```

---

## <a name="validate-the-deployment"></a>Проверка развертывания

Убедитесь, что рабочая область создана с помощью одной из следующих команд. Замените пример значения для **группы ресурсов** значением, которое вы использовали выше.

# <a name="cli"></a>[CLI](#tab/CLI)

```azurecli
az monitor activity-log alert show --resource-group my-resource-group --name AdvisorAlertsTest
```

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

```powershell
Get-AzActivityLogAlert -ResourceGroupName my-resource-group -Name AdvisorAlertsTest
```

---

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

- Изучите [обзор оповещений журнала действий](../azure-monitor/alerts/alerts-overview.md) и узнайте, как получать оповещения.
- Дополнительные сведения о группах действий см. в статье [Создание групп действий и управление ими на портале Azure](../azure-monitor/alerts/action-groups.md).
