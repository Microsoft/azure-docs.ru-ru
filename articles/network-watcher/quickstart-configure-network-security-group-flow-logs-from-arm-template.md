---
title: Краткое руководство. Настройка журналов потоков для группы безопасности сети с помощью шаблона Azure Resource Manager (шаблона ARM)
description: Сведения о том, как включить журналы потоков для группы безопасности сети (NSG) программным способом с помощью шаблона Azure Resource Manager (шаблона ARM) и Azure PowerShell.
services: network-watcher
author: damendo
ms.service: network-watcher
ms.topic: quickstart
ms.date: 01/07/2021
ms.author: damendo
ms.custom: subject-armqs
ms.openlocfilehash: df7f33d9aed307bdf85a571c8169d1b0dbe863e5
ms.sourcegitcommit: 73fb48074c4c91c3511d5bcdffd6e40854fb46e5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/31/2021
ms.locfileid: "106063803"
---
# <a name="quickstart-configure-network-security-group-flow-logs-by-using-an-arm-template"></a>Краткое руководство. Настройка журналов потоков для группы безопасности сети с помощью шаблона Azure Resource Manager

В этом кратком руководстве описано, как включить [журналы потоков для группы безопасности сети (NSG)](network-watcher-nsg-flow-logging-overview.md) с помощью шаблона [Azure Resource Manager](../azure-resource-manager/management/overview.md) (шаблона ARM) и Azure PowerShell.

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Мы начнем с обзора свойств объекта журнала потоков NSG. Будут предоставлены примеры шаблонов. Затем мы будем использовать локальный экземпляр Azure PowerShell для развертывания шаблона.

Если среда соответствует предварительным требованиям и вы знакомы с использованием шаблонов ARM, нажмите кнопку **Развертывание в Azure**. На портале Azure откроется шаблон.

[![Развертывание в Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-networkwatcher-flowLogs-create%2Fazuredeploy.json)

## <a name="prerequisites"></a>Предварительные требования

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

## <a name="review-the-template"></a>Изучение шаблона

Шаблон, используемый в этом кратком руководстве, взят из [шаблонов быстрого запуска Azure](https://azure.microsoft.com/resources/templates/101-networkwatcher-flowlogs-create).

:::code language="json" source="~/quickstart-templates/101-networkwatcher-flowlogs-create/azuredeploy.json":::

В шаблоне определены следующие ресурсы:

- [Microsoft.Storage/storageAccounts](/azure/templates/microsoft.storage/storageaccounts)
- [Microsoft.Resources/deployments](/azure/templates/microsoft.resources/deployments)

## <a name="nsg-flow-logs-object"></a>Объект журналов потоков NSG

В коде ниже показан объект журналов потоков NSG и его параметры. Чтобы создать ресурс `Microsoft.Network/networkWatchers/flowLogs`, добавьте указанный код в раздел ресурсов шаблона:

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

Полный обзор свойств объекта журналов потоков NSG см. в статье [Microsoft.Network networkWatchers/flowLogs](/azure/templates/microsoft.network/networkwatchers/flowlogs).

## <a name="create-your-template"></a>Создание шаблона

Если вы используете шаблоны Resource Manager впервые, ознакомьтесь со следующими статьями, чтобы получить дополнительные сведения:

- [Развертывание ресурсов с помощью шаблонов ARM и Azure PowerShell](../azure-resource-manager/templates/deploy-powershell.md#deploy-local-template-or-bicep-file)
- Создание и развертывание первого шаблона ARM[

В следующем примере приведен полный шаблон. Это также самая простая версия шаблона. Пример содержит минимальные параметры, которые передаются для настройки журналов потоков NSG. Дополнительные примеры см. в обзорной статье [Настройка журналов потоков NSG из шаблона Azure Resource Manager](network-watcher-nsg-flow-logging-azure-resource-manager.md).

### <a name="example"></a>Пример

Следующий шаблон включает журналы потоков для NSG, а затем сохраняет их в определенной учетной записи хранения:

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "apiProfile": "2019-09-01",
  "resources": [
    {
      "name": "NetworkWatcher_centraluseuap/Microsoft.NetworkDalanDemoPerimeterNSG",
      "type": "Microsoft.Network/networkWatchers/FlowLogs/",
      "location": "centraluseuap",
      "apiVersion": "2019-09-01",
      "properties": {
        "targetResourceId": "/subscriptions/<subscription Id>/resourceGroups/DalanDemo/providers/Microsoft.Network/networkSecurityGroups/PerimeterNSG",
        "storageId": "/subscriptions/<subscription Id>/resourceGroups/MyCanaryFlowLog/providers/Microsoft.Storage/storageAccounts/storagev2ira",
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
> - В имени ресурса используется формат _ParentResource_ChildResource_. В нашем примере родительский ресурс является региональным экземпляром Наблюдателя за сетями Azure:
>    - **Format.** NetworkWatcher_RegionName.
>    - **Пример**. NetworkWatcher_centraluseuap.
> - `targetResourceId` является идентификатором ресурса для целевой NSG.
> - `storageId` является идентификатором ресурса для целевой учетной записи хранения.

## <a name="deploy-the-template"></a>Развертывание шаблона

Для выполнения действий, описанных в этом учебнике, предполагается, что у вас есть группа ресурсов и NSG, для которых можно включить ведение журнала потоков.

Вы можете сохранить любой из примеров шаблонов, которые показаны в этой статье, локально в виде файла *azuredeploy.json*. Обновите значения свойств, чтобы они указывали на действительные ресурсы в вашей подписке.

Чтобы развернуть шаблон, выполните следующую команду в PowerShell:

```azurepowershell-interactive
$context = Get-AzSubscription -SubscriptionId <subscription Id>
Set-AzContext $context
New-AzResourceGroupDeployment -Name EnableFlowLog -ResourceGroupName NetworkWatcherRG `
    -TemplateFile "C:\MyTemplates\azuredeploy.json"
```

> [!NOTE]
> Эти команды развертывают ресурс в примере группы ресурсов NetworkWatcherRG, а не в группе ресурсов, содержащей NSG.

## <a name="validate-the-deployment"></a>Проверка развертывания

Существует две возможности проверить успешность развертывания:

- На консоли PowerShell для `ProvisioningState` должно отображаться значение `Succeeded`.
- Вы также можете перейти на [страницу портала журналов потоков NSG](https://ms.portal.azure.com/#blade/Microsoft_Azure_Network/NetworkWatcherMenuBlade/flowLogs), чтобы подтвердить изменения.

Если возникнут проблемы с развертыванием, см. статью [Устранение распространенных ошибок развертывания в Azure с помощью Azure Resource Manager](../azure-resource-manager/templates/common-deployment-errors.md).

## <a name="clean-up-resources"></a>Очистка ресурсов

Вы можете удалить ресурсы Azure с помощью режима полного развертывания. Чтобы удалить ресурс журналов потоков, укажите развертывание в полном режиме, не включая ресурс, который нужно удалить. Ознакомьтесь с дополнительными сведениями о [завершении режима развертывания](../azure-resource-manager/templates/deployment-modes.md#complete-mode).

Вы также можете отключить журнал потоков NSG на портале Azure:

1. Войдите на портал Azure.
1. Выбор пункта **Все службы**. В текстовом поле **Фильтр** введите **Наблюдатель за сетями**. В результатах поиска выберите **Наблюдатель за сетями**.
1. В разделе **Журналы** выберите **Журналы потоков NSG**.
1. В списке групп безопасности сети выберите группу безопасности сети, для которой необходимо отключить журналы потоков.
1. В разделе **Параметры журналов потоков** выберите **Выкл.**
1. Щелкните **Сохранить**.

## <a name="next-steps"></a>Дальнейшие действия

В этом кратком руководстве вы узнали, как включить журналы потоков NSG с помощью шаблона Resource Manager. Далее вы узнаете, как визуализировать данные потока NSG, воспользовавшись одним из следующих вариантов:

- [Microsoft Power BI](network-watcher-visualize-nsg-flow-logs-power-bi.md)
- [Инструменты с открытым кодом](network-watcher-visualize-nsg-flow-logs-open-source-tools.md)
- [Аналитика трафика Azure](traffic-analytics.md)
