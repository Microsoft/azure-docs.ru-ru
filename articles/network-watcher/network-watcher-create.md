---
title: Создание экземпляра Наблюдателя за сетями Azure | Документация Майкрософт
description: Узнайте, как создать наблюдатель за сетями Azure в регионе Azure с помощью портал Azure или других технологий, а также как удалить наблюдатель за сетями.
services: network-watcher
documentationcenter: na
author: damendo
ms.assetid: b1314119-0b87-4f4d-b44c-2c4d0547fb76
ms.service: network-watcher
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/22/2017
ms.author: damendo
ms.openlocfilehash: c308824afdcae5f5c04a316c199bad71ad84a429
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94962014"
---
# <a name="create-an-azure-network-watcher-instance"></a>Создание экземпляра Наблюдателя за сетями Azure

Наблюдатель за сетями — это региональная служба, обеспечивающая мониторинг и диагностику условий на уровне сетевого сценария на платформе Azure. Мониторинг на уровне сценария позволяет диагностировать проблемы в сети с помощью комплексного представления сетевого уровня. Инструменты диагностики сети и визуализации, доступные в Наблюдателе за сетями, помогают понять, как работает сеть в Azure, диагностировать ее и получить ценную информацию. Наблюдатель за сетями включается при создании ресурса "Наблюдатель за сетями". Этот ресурс позволяет использовать возможности Наблюдателя за сетями.


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="network-watcher-is-automatically-enabled"></a>Наблюдатель за сетями включается автоматически
При создании или обновлении виртуальной сети в подписке Наблюдатель за сетями включается автоматически в регионе вашей виртуальной сети. Автоматическое включение Наблюдателя за сетями не влияет на ваши ресурсы, и за него не взимается дополнительная плата.

#### <a name="opt-out-of-network-watcher-automatic-enablement"></a>Отказ от автоматического включения Наблюдателя за сетями
Если вы хотите отказаться от автоматического включения Наблюдателя за сетями, выполните такие команды.

> [!WARNING]
> Отказ от автоматического включения Наблюдателя за сетями — действие необратимое. Чтобы возобновить автоматическое включение после отказа от него, придется [обратиться в службу поддержки](https://azure.microsoft.com/support/options/).

```azurepowershell-interactive
Register-AzProviderFeature -FeatureName DisableNetworkWatcherAutocreation -ProviderNamespace Microsoft.Network
Register-AzResourceProvider -ProviderNamespace Microsoft.Network
```

```azurecli-interactive
az feature register --name DisableNetworkWatcherAutocreation --namespace Microsoft.Network
az provider register -n Microsoft.Network
```



## <a name="create-a-network-watcher-in-the-portal"></a>Создание Наблюдателя за сетями на портале

Последовательно выберите **все службы**  >  **сеть**  >  **наблюдатель за сетями**. Можно выбрать все подписки, для которых необходимо включить Наблюдатель за сетями. Это действие создаст экземпляр Наблюдателя за сетями в каждом регионе, который доступен.

![Создание Наблюдателя за сетями](./media/network-watcher-create/figure1.png)

Когда вы включаете Наблюдатель за сетями с помощью портала, созданному экземпляру наблюдателя автоматически присваивается имя *NetworkWatcher_имя_региона*, где *имя_региона* соответствует региону Azure, в котором был включен этот экземпляр наблюдателя. Например, включенный в центрально-западной части США Наблюдатель за сетями получит имя *NetworkWatcher_westcentralus*.

Экземпляр Наблюдателя за сетями автоматически создается в группе ресурсов с именем *NetworkWatcherRG*. Если эта группа ресурсов не существует, она создается.

Если вы хотите настроить имя экземпляра наблюдателя за сетями и группу ресурсов, в которую он помещен, можно использовать PowerShell, Azure CLI, REST API или методы ARMClient, описанные в следующих разделах. Во всех этих вариантах должна заранее существовать группа ресурсов, в которой вы создаете Наблюдатель за сетями.  

## <a name="create-a-network-watcher-with-powershell"></a>Создание Наблюдателя за сетями с помощью PowerShell

Чтобы создать экземпляр Наблюдателя за сетями, выполните приведенный ниже пример.

```powershell
New-AzNetworkWatcher -Name "NetworkWatcher_westcentralus" -ResourceGroupName "NetworkWatcherRG" -Location "West Central US"
```

## <a name="create-a-network-watcher-with-the-azure-cli"></a>Создание Наблюдателя за сетями с помощью Azure CLI

Чтобы создать экземпляр Наблюдателя за сетями, выполните приведенный ниже пример.

```azurecli
az network watcher configure --resource-group NetworkWatcherRG --locations westcentralus --enabled
```

## <a name="create-a-network-watcher-with-the-rest-api"></a>Создание Наблюдателя за сетями с помощью REST API

Чтобы вызвать REST API с помощью командлетов PowerShell, вам потребуется ARMClient. Пакет ARMClient можно скачать на сайте [Chocolatey](https://chocolatey.org/packages/ARMClient).

### <a name="log-in-with-armclient"></a>выполните вход с помощью ARMClient;

```powerShell
armclient login
```

### <a name="create-the-network-watcher"></a>Создание Наблюдателя за сетями

```powershell
$subscriptionId = '<subscription id>'
$networkWatcherName = '<name of network watcher>'
$resourceGroupName = '<resource group name>'
$apiversion = "2016-09-01"
$requestBody = @"
{
'location': 'West Central US'
}
"@

armclient put "https://management.azure.com/subscriptions/${subscriptionId}/resourceGroups/${resourceGroupName}/providers/Microsoft.Network/networkWatchers/${networkWatcherName}?api-version=${api-version}" $requestBody
```

## <a name="create-a-network-watcher-using-azure-quickstart-template"></a>Создание наблюдателя за сетями с помощью шаблона быстрого запуска Azure

Чтобы создать экземпляр наблюдателя за сетями, обратитесь к этому [шаблону краткого руководства](https://azure.microsoft.com/resources/templates/101-networkwatcher-create/)

## <a name="delete-a-network-watcher-in-the-portal"></a>Удаление наблюдателя за сетями на портале

Последовательно выберите **все службы**  >  **сеть**  >  **наблюдатель за сетями**.

Перейдите на вкладку Обзор, если она еще не установлена. В раскрывающемся списке выберите подписку, для которой нужно отключить наблюдатель за сетями.
Разверните список регионов для выбранной подписки, щелкнув стрелку. Для любого из этих данных используйте три точки справа для доступа к контекстному меню.
Щелкните "отключить наблюдатель за сетями", чтобы начать отключение. Вам будет предложено подтвердить этот шаг. Для продолжения нажмите кнопку Да.
На портале это необходимо сделать отдельно для каждого региона в каждой подписке.


## <a name="delete-a-network-watcher-with-powershell"></a>Удаление наблюдателя за сетями с помощью PowerShell

Чтобы удалить экземпляр наблюдателя за сетями, выполните следующий пример:

```powershell
New-AzResourceGroup -Name NetworkWatcherRG -Location westcentralus
New-AzNetworkWatcher -Name NetworkWatcher_westcentralus -ResourceGroup NetworkWatcherRG -Location westcentralus
Remove-AzNetworkWatcher -Name NetworkWatcher_westcentralus -ResourceGroup NetworkWatcherRG
```

## <a name="next-steps"></a>Дальнейшие действия

Теперь, имея экземпляр Наблюдателя за сетями, узнайте о его возможностях:

* [Топология](./view-network-topology.md)
* [Запись пакетов](network-watcher-packet-capture-overview.md)
* [Проверка IP-потока](network-watcher-ip-flow-verify-overview.md)
* [Следующий прыжок](network-watcher-next-hop-overview.md)
* [Представление групп безопасности](network-watcher-security-group-view-overview.md)
* [Ведение журнала потоков NSG](network-watcher-nsg-flow-logging-overview.md)
* [Устранение неполадок шлюза виртуальной сети](network-watcher-troubleshoot-overview.md)