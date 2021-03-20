---
title: Запрос к подпискам службы "Сетка событий Azure"
description: В этой статье описывается, как получить список подписок на сетку событий в подписке Azure. Вы предоставляете различные параметры в зависимости от типа подписки.
ms.topic: conceptual
ms.date: 07/07/2020
ms.openlocfilehash: 3d700f543bc5e3c7add2a346c10acf975e1c2462
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "86120455"
---
# <a name="query-event-grid-subscriptions"></a>Запрос к подпискам службы "Сетка событий Azure" 

В этой статье описано, как получить список подписок Сетки событий для подписки Azure. При создании запросов к существующим подпискам Сетки событий важно понимать различия между типами подписок. Параметры запросов будут зависеть от требуемого типа подписки.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="resource-groups-and-azure-subscriptions"></a>Группы ресурсов и подписки Azure

Подписки и группы ресурсов Azure не являются ресурсами Azure. Это означает, что подписки Сетки событий на группы ресурсов или подписки Azure будут иметь другие свойства, отличные от свойств для подписок Сетки событий на ресурсы Azure. Подписки Сетки событий на группы ресурсов или подписки Azure считаются глобальными.

Чтобы получить подписки Сетки событий для группы ресурсов или подписки Azure, не нужно указывать никаких параметров. Достаточно выбрать подписку Azure, к которой вы направляете запрос. Следующие примеры не возвращают подписки Сетки событий для настраиваемых разделов и (или) ресурсов Azure.

Для интерфейса командной строки Azure:

```azurecli-interactive
az account set -s "My Azure Subscription"
az eventgrid event-subscription list
```

Для PowerShell используйте команду:

```azurepowershell-interactive
Set-AzContext -Subscription "My Azure Subscription"
Get-AzEventGridSubscription
```

Чтобы получить подписки Сетки событий для подписки Azure, укажите тип раздела **Microsoft.Resources.Subscriptions**.

Для интерфейса командной строки Azure:

```azurecli-interactive
az eventgrid event-subscription list --topic-type-name "Microsoft.Resources.Subscriptions" --location global
```

Для PowerShell используйте команду:

```azurepowershell-interactive
Get-AzEventGridSubscription -TopicTypeName "Microsoft.Resources.Subscriptions"
```

Чтобы получить подписки Сетки событий для всех групп ресурсов в подписке Azure, укажите тип раздела **Microsoft.Resources.ResourceGroups**.

Для интерфейса командной строки Azure:

```azurecli-interactive
az eventgrid event-subscription list --topic-type-name "Microsoft.Resources.ResourceGroups" --location global
```

Для PowerShell используйте команду:

```azurepowershell-interactive
Get-AzEventGridSubscription -TopicTypeName "Microsoft.Resources.ResourceGroups"
```

Чтобы получить подписки Сетки событий для определенной группы ресурсов, укажите имя этой группы ресурсов в качестве параметра.

Для интерфейса командной строки Azure:

```azurecli-interactive
az eventgrid event-subscription list --resource-group myResourceGroup --location global
```

Для PowerShell используйте команду:

```azurepowershell-interactive
Get-AzEventGridSubscription -ResourceGroupName myResourceGroup
```

## <a name="custom-topics-and-azure-resources"></a>Настраиваемые разделы и ресурсы Azure

Настраиваемые разделы Сетки событий являются ресурсами Azure. Это означает, что можно использовать одинаковые запросы, чтобы получить подписки Сетки событий сетки для пользовательских разделов и других ресурсов, например для учетной записи хранилища больших двоичных объектов. Чтобы получить подписки Сетки событий для настраиваемых разделов, необходимо указать параметры, однозначно определяющие ресурс или расположение ресурса. Вы не можете с помощью одного запроса получить подписки Сетки событий для всех ресурсов в подписке Azure.

Чтобы получить подписки Сетки событий для настраиваемых разделов и других ресурсов в определенном расположении, укажите имя этого расположения.

Для интерфейса командной строки Azure:

```azurecli-interactive
az eventgrid event-subscription list --location westus2
```

Для PowerShell используйте команду:

```azurepowershell-interactive
Get-AzEventGridSubscription -Location westus2
```

Чтобы получить подписки на настраиваемые разделы в определенном расположении, укажите это расположение и тип раздела **Microsoft.EventGrid.Topics**.

Для интерфейса командной строки Azure:

```azurecli-interactive
az eventgrid event-subscription list --topic-type-name "Microsoft.EventGrid.Topics" --location "westus2"
```

Для PowerShell используйте команду:

```azurepowershell-interactive
Get-AzEventGridSubscription -TopicTypeName "Microsoft.EventGrid.Topics" -Location westus2
```

Чтобы получить подписки на учетные записи хранения в определенном расположении, укажите это расположение и тип раздела **Microsoft.Storage.StorageAccounts**.

Для интерфейса командной строки Azure:

```azurecli-interactive
az eventgrid event-subscription list --topic-type "Microsoft.Storage.StorageAccounts" --location westus2
```

Для PowerShell используйте команду:

```azurepowershell-interactive
Get-AzEventGridSubscription -TopicTypeName "Microsoft.Storage.StorageAccounts" -Location westus2
```

Чтобы получить подписки Сетки событий на пользовательский раздел, укажите имя этого раздела и имя соответствующей группы ресурсов.

Для интерфейса командной строки Azure:

```azurecli-interactive
az eventgrid event-subscription list --topic-name myCustomTopic --resource-group myResourceGroup
```

Для PowerShell используйте команду:

```azurepowershell-interactive
Get-AzEventGridSubscription -TopicName myCustomTopic -ResourceGroupName myResourceGroup
```

Чтобы получить подписки Сетки событий для определенного ресурса, укажите идентификатор этого ресурса.

Для интерфейса командной строки Azure:

```azurecli-interactive
resourceid=$(az resource show -n mystorage -g myResourceGroup --resource-type "Microsoft.Storage/storageaccounts" --query id --output tsv)
az eventgrid event-subscription list --resource-id $resourceid
```

Для PowerShell используйте команду:

```azurepowershell-interactive
$resourceid = (Get-AzResource -Name mystorage -ResourceGroupName myResourceGroup).ResourceId
Get-AzEventGridSubscription -ResourceId $resourceid
```

## <a name="next-steps"></a>Дальнейшие действия

* См. дополнительные сведения о [доставке сообщений и повторных попытках в Сетке событий](delivery-and-retry.md).
* Общие сведения о службе "Сетка событий" см. в разделе [Общие сведения о службе "Сетка событий Azure"](overview.md).
* Сведения о том, как быстро приступить к использованию службы "Сетка событий", см. в разделе [Создание и перенаправление пользовательского события со службой "Сетка событий Azure"](custom-event-quickstart.md).
