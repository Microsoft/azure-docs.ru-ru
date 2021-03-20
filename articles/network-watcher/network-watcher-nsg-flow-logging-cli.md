---
title: Управление журналами потоков NSG — Azure CLI
titleSuffix: Azure Network Watcher
description: На этой странице объясняется, как управлять журналами потоков для групп безопасности сети с помощью Наблюдателя за сетями в Azure CLI
services: network-watcher
documentationcenter: na
author: damendo
ms.service: network-watcher
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 01/07/2021
ms.author: damendo
ms.openlocfilehash: 46d12db413fdf01995bc84ae018065e877afb15e
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98017822"
---
# <a name="configuring-network-security-group-flow-logs-with-azure-cli"></a>Настройка журналов потоков для групп безопасности сети с помощью Azure CLI

> [!div class="op_single_selector"]
> - [Портал Azure](network-watcher-nsg-flow-logging-portal.md)
> - [PowerShell](network-watcher-nsg-flow-logging-powershell.md)
> - [Azure CLI](network-watcher-nsg-flow-logging-cli.md)
> - [REST API](network-watcher-nsg-flow-logging-rest.md)

Журналы потоков для групп безопасности сети — это компонент Наблюдателя за сетями, который позволяет просматривать сведения о входящем и исходящем IP-трафике через группу безопасности сети. Эти журналы потоков записываются в формате JSON. В них отображаются входящие и исходящие потоки по каждому правилу, сетевая карта, с которой связан поток, сведения о 5 кортежах потока (IP-адрес источника и места назначения, порт источника и места назначения, протокол), а также сведения о состоянии трафика (разрешен или запрещен).

Для выполнения действий, описанных в этой статье, требуется [установить Azure CLI для Mac, Linux и Windows](/cli/azure/install-azure-cli).

## <a name="register-insights-provider"></a>Регистрация поставщика Microsoft Insights

Для успешного ведения журналов потоков должен быть зарегистрирован поставщик **Microsoft.Insights**. Если вы не знаете, зарегистрирован ли поставщик **Microsoft.Insights**, выполните следующий сценарий.

```azurecli
az provider register --namespace Microsoft.Insights
```

## <a name="enable-network-security-group-flow-logs"></a>Включение журналов потоков для группы безопасности сети

Команда для включения журналов потоков показана в следующем примере.

```azurecli
az network watcher flow-log create --resource-group resourceGroupName --enabled true --nsg nsgName --storage-account storageAccountName --location location
# Configure 
az network watcher flow-log create --resource-group resourceGroupName --enabled true --nsg nsgName --storage-account storageAccountName --location location --format JSON --log-version 2
```

В указанной учетной записи хранения не может быть настроенных сетевых правил, ограничивающих доступ к сети только службами Майкрософт или конкретными виртуальными сетями. Учетная запись хранения может быть в той же подписке Azure, что и NSG, для которой включается журнал потока, или в другой подписке Azure. Если используются разные подписки, они должны быть связаны с одним клиентом Azure Active Directory. Используемая учетная запись для каждой подписки должна иметь [необходимые разрешения](required-rbac-permissions.md). 

Если учетная запись хранения находится не в той же группе ресурсов или подписке, где и группа сетевой безопасности, укажите полный идентификатор учетной записи хранения, а не ее имя. Например, если учетная запись хранения находится в группе ресурсов с именем *RG-Storage*, вместо того чтобы в предыдущей команде задавать *storageAccountName*, следует указать */subscriptions/{SubscriptionID}/resourceGroups/RG-Storage/providers/Microsoft.Storage/storageAccounts/storageAccountName*.

## <a name="disable-network-security-group-flow-logs"></a>Отключение журналов потоков для группы безопасности сети

Чтобы отключить журналы потоков, используйте следующий пример:

```azurecli
az network watcher flow-log configure --resource-group resourceGroupName --enabled false --nsg nsgName
```

## <a name="download-a-flow-log"></a>Скачивание журнала потоков

Место хранения журнала потоков определяется при его создании. Удобное средство для доступа к этим журналам потоков, сохраненным в учетной записи хранения, — обозреватель службы хранилища Microsoft Azure, который можно скачать по адресу https://storageexplorer.com/

При указании учетной записи хранения файлы журнала потоков сохраняются в ней по следующему адресу:

```
https://{storageAccountName}.blob.core.windows.net/insights-logs-networksecuritygroupflowevent/resourceId=/SUBSCRIPTIONS/{subscriptionID}/RESOURCEGROUPS/{resourceGroupName}/PROVIDERS/MICROSOFT.NETWORK/NETWORKSECURITYGROUPS/{nsgName}/y={year}/m={month}/d={day}/h={hour}/m=00/macAddress={macAddress}/PT1H.json
```


## <a name="next-steps"></a>Дальнейшие действия

Узнайте, как [визуализировать сведения журналов потоков группы безопасности сети с помощью PowerBI](network-watcher-visualize-nsg-flow-logs-power-bi.md).

Узнайте, как [визуализировать сведения журналов потоков группы безопасности сети с помощью средств с открытым кодом](network-watcher-visualize-nsg-flow-logs-open-source-tools.md).
