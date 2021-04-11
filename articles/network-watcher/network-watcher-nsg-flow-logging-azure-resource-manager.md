---
title: Наблюдатель за сетями. Создание журналов потоков NSG с помощью шаблона Azure Resource Manager
description: Используйте шаблон Azure Resource Manager и PowerShell, чтобы без усилий настроить журналы потоков NSG.
services: network-watcher
documentationcenter: na
author: damendo
manager: twooley
editor: ''
tags: azure-resource-manager
ms.service: network-watcher
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 01/07/2021
ms.author: damendo
ms.custom: fasttrack-edit
ms.openlocfilehash: 2cae673dd319e55670357ae468d21ff63e4f9c72
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "104669446"
---
# <a name="configure-nsg-flow-logs-from-an-azure-resource-manager-template"></a>Настройка журналов потоков NSG из шаблона Azure Resource Manager

> [!div class="op_single_selector"]
> - [Портал Azure](network-watcher-nsg-flow-logging-portal.md)
> - [PowerShell](network-watcher-nsg-flow-logging-powershell.md)
> - [Azure CLI](network-watcher-nsg-flow-logging-cli.md)
> - [REST API](network-watcher-nsg-flow-logging-rest.md)
> - [Azure Resource Manager](network-watcher-nsg-flow-logging-azure-resource-manager.md)


[Azure Resource Manager](https://azure.microsoft.com/features/resource-manager/) — это нативный эффективный способ управления [инфраструктурой как кодом](/azure/devops/learn/what-is-infrastructure-as-code).

В этой статье показано, как включить [журналы потоков NSG](./network-watcher-nsg-flow-logging-overview.md) программным способом с помощью шаблона Azure Resource Manager и Azure PowerShell. Сначала мы рассмотрим свойства объекта журнала потоков NSG, а также несколько примеров шаблонов. Затем мы развернем шаблон с помощью локального экземпляра PowerShell.


## <a name="nsg-flow-logs-object"></a>Объект журналов потоков NSG

Ниже показан объект журналов потоков NSG со всеми параметрами.
Полный обзор свойств см. в [справочнике по шаблонам журналов потоков NSG](/azure/templates/microsoft.network/2019-11-01/networkwatchers/flowlogs#RetentionPolicyParameters).

```json
{
  "name": "string",
  "type": "Microsoft.Network/networkWatchers/flowLogs",
  "location": "string",
  "apiVersion": "2019-09-01",
  "properties": {
    "targetResourceId": "string",
    "storageId": "string",
    "enabled": "boolean",
    "flowAnalyticsConfiguration": {
      "networkWatcherFlowAnalyticsConfiguration": {
         "enabled": "boolean",
         "workspaceResourceId": "string",
          "trafficAnalyticsInterval": "integer"
        },
        "retentionPolicy": {
           "days": "integer",
           "enabled": "boolean"
         },
        "format": {
           "type": "string",
           "version": "integer"
         }
      }
    }
  }
```
Чтобы создать ресурс Microsoft.Network/networkWatchers/flowLogs, добавьте приведенный выше код JSON в раздел ресурсов шаблона.


## <a name="creating-your-template"></a>Создание шаблона

Если вы используете шаблоны Azure Resource Manager в первый раз, вы можете узнать больше о них, перейдя по указанным ниже ссылкам.

* [Развертывание ресурсов с использованием шаблонов Resource Manager и Azure PowerShell](../azure-resource-manager/templates/deploy-powershell.md#deploy-local-template-or-bicep-file)
* [Руководство. Создание и развертывание первого шаблона Azure Resource Manager](../azure-resource-manager/templates/template-tutorial-create-first-template.md?tabs=azure-powershell)


Ниже приведены два примера готовых шаблонов для настройки журналов потоков NSG.

**Пример 1**:  простейшая версия с минимальным числом переданных параметров. Приведенный ниже шаблон включает журналы потоков NSG на целевом NSG и сохраняет их в определенной учетной записи хранения.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "apiProfile": "2019-09-01",
  "resources": [
 {
    "name": "NetworkWatcher_centraluseuap/Microsoft.NetworkDalanDemoPerimeterNSG",
    "type": "Microsoft.Network/networkWatchers/FlowLogs/",
    "location": "centraluseuap",
    "apiVersion": "2019-09-01",
    "properties": {
      "targetResourceId": "/subscriptions/56abfbd6-ec72-4ce9-831f-bc2b6f2c5505/resourceGroups/DalanDemo/providers/Microsoft.Network/networkSecurityGroups/PerimeterNSG",
      "storageId": "/subscriptions/56abfbd6-ec72-4ce9-831f-bc2b6f2c5505/resourceGroups/MyCanaryFlowLog/providers/Microsoft.Storage/storageAccounts/storagev2ira",
      "enabled": true,
      "flowAnalyticsConfiguration": {},
      "retentionPolicy": {},
      "format": {}
    }

  }
  ]
}
```

> [!NOTE]
> * Имя ресурса имеет формат "родительский ресурс_дочерний ресурс". Здесь родительский ресурс является региональным экземпляром Наблюдателя за сетями (формат: NetworkWatcher_RegionName. Пример NetworkWatcher_centraluseuap).
> * targetResourceId — идентификатор ресурса целевой группы NSG.
> * storageId — это идентификатор ресурса целевой учетной записи хранения.

**Пример 2**. Следующие шаблоны позволяют использовать журналы потоков NSG (версия 2) с периодом хранения в 5 дней. Включение Аналитики трафика с интервалом обработки в 10 минут.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "apiProfile": "2019-09-01",
  "resources": [
    {
      "name": "NetworkWatcher_centraluseuap/Microsoft.NetworkDalanDemoPerimeterNSG",
      "type": "Microsoft.Network/networkWatchers/FlowLogs/",
      "location": "centraluseuap",
      "apiVersion": "2019-09-01",
      "properties": {
        "targetResourceId": "/subscriptions/56abfbd6-ec72-4ce9-831f-bc2b6f2c5505/resourceGroups/DalanDemo/providers/Microsoft.Network/networkSecurityGroups/PerimeterNSG",
        "storageId": "/subscriptions/56abfbd6-ec72-4ce9-831f-bc2b6f2c5505/resourceGroups/MyCanaryFlowLog/providers/Microsoft.Storage/storageAccounts/storagev2ira",
        "enabled": true,
        "flowAnalyticsConfiguration": {
          "networkWatcherFlowAnalyticsConfiguration": {
            "enabled": true,
            "workspaceResourceId": "/subscriptions/56abfbd6-ec72-4ce9-831f-bc2b6f2c5505/resourceGroups/defaultresourcegroup-wcus/providers/Microsoft.OperationalInsights/workspaces/1c4f42e5-3a02-4146-ac9b-3051d8501db0",
            "trafficAnalyticsInterval": 10
          }
        },
        "retentionPolicy": {
          "days": 5,
          "enabled": true
        },
        "format": {
          "type": "JSON",
          "version": 2
        }
      }
    }
  ]
}
```

## <a name="deploying-your-azure-resource-manager-template"></a>Развертывание шаблона Azure Resource Manager

В этом руководстве предполагается, что у вас есть группа ресурсов и NSG, для которых можно включить ведение журнала потоков.
Любой из приведенных выше примеров шаблонов можно сохранить локально как `azuredeploy.json`. Обновите значения свойств, чтобы они указывали на действительные ресурсы в вашей подписке.

Чтобы развернуть шаблон, выполните следующую команду в PowerShell.
```azurepowershell
$context = Get-AzSubscription -SubscriptionId <SubscriptionId>
Set-AzContext $context
New-AzResourceGroupDeployment -Name EnableFlowLog -ResourceGroupName NetworkWatcherRG `
    -TemplateFile "C:\MyTemplates\azuredeploy.json"
```

> [!NOTE]
> Приведенные выше команды развертывают ресурс в группе ресурсов NetworkWatcherRG, а не в группе ресурсов, содержащей NSG.


## <a name="verifying-your-deployment"></a>Проверка развертывания

Существует несколько способов проверить, успешно ли выполнено развертывание. В консоли PowerShell для параметра ProvisioningState должно отобразиться значения Succeeded. Кроме того, вы можете посетить страницу [журналов потоков NSG на портале](https://ms.portal.azure.com/#blade/Microsoft_Azure_Network/NetworkWatcherMenuBlade/flowLogs), чтобы подтвердить изменения. Если возникнут проблемы с развертыванием, см. статью [Устранение распространенных ошибок развертывания в Azure с помощью Azure Resource Manager](../azure-resource-manager/templates/common-deployment-errors.md).

## <a name="deleting-your-resource"></a>Удаление ресурса
Azure позволяет удалять ресурсы с помощью полного режима развертывания. Чтобы удалить ресурс журналов потоков, укажите развертывание в полном режиме, не включая ресурс, который нужно удалить. См. сведения о [полном режиме развертывания](../azure-resource-manager/templates/deployment-modes.md#complete-mode).

## <a name="next-steps"></a>Дальнейшие действия

Узнайте, как визуализировать данные NSG Flow с помощью следующих средств:
* [Microsoft Power BI](network-watcher-visualize-nsg-flow-logs-power-bi.md)
* [Средства с открытым кодом](network-watcher-visualize-nsg-flow-logs-open-source-tools.md)
* [Аналитика трафика Azure](./traffic-analytics.md)
