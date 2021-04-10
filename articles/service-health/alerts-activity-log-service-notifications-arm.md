---
title: Получение оповещений журнала действий для уведомлений службы Azure с помощью шаблона Resource Manager
description: Получайте уведомления по SMS, электронной почте или от веб-перехватчика при использовании службы Azure.
ms.topic: quickstart
ms.custom: subject-armqs
ms.date: 06/29/2020
ms.openlocfilehash: 532fbae505e0bcaa6ab31a2e935362114537d134
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "100594954"
---
# <a name="quickstart-create-activity-log-alerts-on-service-notifications-using-an-arm-template"></a>Краткое руководство. Создание оповещений журнала действий для уведомлений службы с помощью шаблона Resource Manager

В этой статье показано, как настроить оповещения журнала действий для уведомлений о работоспособности службы с помощью шаблона Resource Manager.

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Уведомления о работоспособности службы сохраняются в [журнале действий Azure](../azure-monitor/essentials/platform-logs-overview.md). При наличии большого объема информации, хранящегося в журнале действий, вы можете воспользоваться отдельным пользовательским интерфейсом, который упрощает просмотр и настройку оповещений о работоспособности службы.

Оповещения можно получать, когда Azure отправляет уведомления о работоспособности службы в вашу подписку Azure. Оповещения можно настроить на основе следующих данных.

- Класс уведомления о работоспособности службы (проблемы со службой, запланированное обслуживание, рекомендации по работоспособности).
- Затронутая подписка.
- Затронутые службы.
- Затронутые регионы.

> [!NOTE]
> Уведомления о работоспособности службы не содержат предупреждения о событиях работоспособности ресурсов.

Кроме того, можно настроить, кому должны отправляться оповещения:

- Выберите имеющуюся группу действий.
- Создайте группу действий (которую можно будет использовать для будущих оповещений).

Дополнительные сведения о группах действий см. в разделе [Создание групп действий и управление ими на портале Azure](../azure-monitor/alerts/action-groups.md).

## <a name="prerequisites"></a>Предварительные требования

- Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.
- Чтобы выполнять команды с локального компьютера, установите Azure CLI или модули Azure PowerShell. Дополнительные сведения см. в статьях [Установка Azure CLI](/cli/azure/install-azure-cli) и [Установка Azure PowerShell](/powershell/azure/install-az-ps).

## <a name="review-the-template"></a>Изучение шаблона

Ниже приведен шаблон, который создает группу действий с целевым объектом электронной почты и активирует все уведомления о работоспособности службы в целевой подписке. Сохраните этот шаблон как файл *CreateServiceHealthAlert.json*.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "actionGroups_name": {
      "type": "String",
      "defaultValue": "SubHealth"
    },
    "activityLogAlerts_name": {
      "type": "String",
      "defaultValue": "ServiceHealthActivityLogAlert"
    },
    "emailAddress": {
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
      "comments": "Service Health Activity Log Alert",
      "type": "microsoft.insights/activityLogAlerts",
      "apiVersion": "2017-04-01",
      "name": "[parameters('activityLogAlerts_name')]",
      "location": "Global",
      "scale": null,
      "dependsOn": [
        "[resourceId('microsoft.insights/actionGroups', parameters('actionGroups_name'))]"
      ],
      "tags": {},
      "properties": {
        "scopes": [
          "[variables('alertScope')]"
        ],
        "condition": {
          "allOf": [
            {
              "field": "category",
              "equals": "ServiceHealth"
            },
            {
              "field": "properties.incidentType",
              "equals": "Incident"
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
      }
    }
  ]
}
```

Шаблон определяет два ресурса:

- [Microsoft.Insights/actionGroups](/azure/templates/microsoft.insights/actiongroups)
- [Microsoft.Insights/activityLogAlerts](/azure/templates/microsoft.insights/activityLogAlerts)

## <a name="deploy-the-template"></a>Развертывание шаблона

Разверните шаблон с помощью любого стандартного метода [развертывания шаблона ARM](../azure-resource-manager/templates/deploy-portal.md), например с помощью CLI и PowerShell, как в примерах ниже. Замените примеры значений для **группы ресурсов** и **emailAddress** соответствующими значениями для вашей среды.

# <a name="cli"></a>[CLI](#tab/CLI)

```azurecli
az login
az deployment group create --name CreateServiceHealthAlert --resource-group my-resource-group --template-file CreateServiceHealthAlert.json --parameters emailAddress='user@contoso.com'
```

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

```powershell
Connect-AzAccount
Select-AzSubscription -SubscriptionName my-subscription
New-AzResourceGroupDeployment -Name CreateServiceHealthAlert -ResourceGroupName my-resource-group -TemplateFile CreateServiceHealthAlert.json -emailAddress user@contoso.com
```

---

## <a name="validate-the-deployment"></a>Проверка развертывания

Убедитесь, что рабочая область создана с помощью одной из следующих команд. Замените пример значения для **группы ресурсов** значением, которое вы использовали выше.

# <a name="cli"></a>[CLI](#tab/CLI)

```azurecli
az monitor activity-log alert show --resource-group my-resource-group --name ServiceHealthActivityLogAlert
```

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

```powershell
Get-AzActivityLogAlert -ResourceGroupName my-resource-group -Name ServiceHealthActivityLogAlert
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

- Ознакомьтесь с [рекомендациями по настройке оповещений Работоспособности служб Azure](https://www.microsoft.com/en-us/videoplayer/embed/RE2OtUa).
- Узнайте, как [настроить мобильные push-уведомления для Работоспособности служб Azure](https://www.microsoft.com/en-us/videoplayer/embed/RE2OtUw).
- Узнайте, [как настроить уведомления веб-перехватчика для существующих систем управления проблемами](service-health-alert-webhook-guide.md).
- Дополнительные сведения об уведомлениях о работоспособности службы см. в [этой статье](service-notifications.md).
- Дополнительные сведения об ограничении частоты отправки уведомлений см. в статье [Ограничение частоты отправки для SMS, сообщений электронной почты и вызовов Webhook](../azure-monitor/alerts/alerts-rate-limiting.md).
- Просмотрите схему веб-перехватчика оповещений журнала действий в статье [Объекты webhook для оповещений журнала действий Azure](../azure-monitor/alerts/activity-log-alerts-webhook.md).
- Изучите [обзор оповещений журнала действий](../azure-monitor/alerts/alerts-overview.md) и узнайте, как получать оповещения.
- Дополнительные сведения о группах действий см. в статье [Создание групп действий и управление ими на портале Azure](../azure-monitor/alerts/action-groups.md).
