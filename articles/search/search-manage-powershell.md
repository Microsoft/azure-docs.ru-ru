---
title: Сценарии PowerShell с помощью команды AZ. Search
titleSuffix: Azure Cognitive Search
description: Создание и Настройка службы Когнитивный поиск Azure с помощью PowerShell. Вы можете масштабировать службу по мере увеличения или уменьшения масштаба, управлять ключами API администрирования и запросов, а также запрашивать сведения о системе.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.devlang: powershell
ms.topic: conceptual
ms.date: 02/09/2021
ms.openlocfilehash: efb5d498c627a6731d2a90623c81eefabd0042a0
ms.sourcegitcommit: afb9e9d0b0c7e37166b9d1de6b71cd0e2fb9abf5
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/14/2021
ms.locfileid: "103462785"
---
# <a name="manage-your-azure-cognitive-search-service-with-powershell"></a>Управление службой Когнитивный поиск Azure с помощью PowerShell
> [!div class="op_single_selector"]
> * [Портал](search-manage.md)
> * [PowerShell](search-manage-powershell.md)
> * [Azure CLI](search-manage-azure-cli.md)
> * [REST API](/rest/api/searchmanagement/)
> * [Пакет SDK для .NET](/dotnet/api/microsoft.azure.management.search)
> * [Python](https://pypi.python.org/pypi/azure-mgmt-search/0.1.0)

Вы можете запускать командлеты и скрипты PowerShell в Windows, Linux или в [Azure Cloud Shell](../cloud-shell/overview.md) для создания и настройки когнитивный Поиск Azure. Модуль **AZ. Search** расширяет [Azure PowerShell](/powershell/) с полным контролем четности к [API-интерфейсам управления поиском](/rest/api/searchmanagement) и позволяет выполнять следующие задачи:

> [!div class="checklist"]
> * [Перечисление служб поиска в подписке](#list-search-services)
> * [Возврат сведений о службе](#get-search-service-information)
> * [Создание или удаление службы](#create-or-delete-a-service)
> * [Создание службы с закрытой конечной точкой](#create-a-service-with-a-private-endpoint)
> * [Повторное создание ключей API администратора](#regenerate-admin-keys)
> * [Создание или удаление ключей API запроса](#create-or-delete-query-keys)
> * [Увеличение или уменьшение масштаба с помощью реплик и секций](#scale-replicas-and-partitions)
> * [Создание общего ресурса частной ссылки](#create-a-shared-private-link-resource)

Иногда вопросы задаются *не* в приведенном выше списке. В настоящее время нельзя использовать модуль **AZ. Search** или REST API управления для изменения имени сервера, региона или уровня. Выделенные ресурсы выделяются при создании службы. Таким образом, для изменения базового оборудования (расположения или типа узла) требуется новая служба. Аналогично, отсутствуют средства или интерфейсы API для передачи содержимого, например индекса, из одной службы в другую.

В рамках службы создание и управление содержимым осуществляется с помощью [Служба поиска REST API](/rest/api/searchservice/) или [пакета SDK для .NET](/dotnet/api/overview/azure/search.documents-readme). Хотя для содержимого нет выделенных команд PowerShell, можно написать сценарий PowerShell, который вызывает API-интерфейсы RESTFUL или .NET для создания и загрузки индексов.

<a name="check-versions-and-load"></a>

## <a name="check-versions-and-load-modules"></a>Проверка версий и модулей загрузки

Примеры в этой статье являются интерактивными и нуждаются в повышенных правах. Необходимо установить Azure PowerShell (модуль **AZ** ). Дополнительные сведения см. в статье [Установка Azure PowerShell](/powershell/azure/).

### <a name="powershell-version-check-51-or-later"></a>Проверка версии PowerShell (5,1 или более поздней версии)

Локальное PowerShell должно быть 5,1 или более поздней версии в любой поддерживаемой операционной системе.

```azurepowershell-interactive
$PSVersionTable.PSVersion
```

### <a name="load-azure-powershell"></a>Azure PowerShell загрузки

Если вы не уверены, установлено ли **AZ** , выполните следующую команду в качестве шага проверки. 

```azurepowershell-interactive
Get-InstalledModule -Name Az
```

Некоторые системы не выполняют автоматическую загрузку модулей. Если при выполнении предыдущей команды возникнет ошибка, попробуйте загрузить модуль. Если это не удается, вернитесь к инструкциям по установке, чтобы узнать, пропущен ли шаг.

```azurepowershell-interactive
Import-Module -Name Az
```

### <a name="connect-to-azure-with-a-browser-sign-in-token"></a>Подключение к Azure с помощью токена входа в браузере

Для подключения к подписке в PowerShell можно использовать учетные данные для входа на портал. Кроме того, можно [выполнять проверку подлинности в неинтерактивном режиме с помощью субъекта-службы](../active-directory/develop/howto-authenticate-service-principal-powershell.md).

```azurepowershell-interactive
Connect-AzAccount
```

Если вы удерживаете несколько подписок Azure, задайте свою подписку Azure. Чтобы просмотреть список текущих подписок, выполните следующую команду.

```azurepowershell-interactive
Get-AzSubscription | sort SubscriptionName | Select SubscriptionName
```

Чтобы указать подписку, выполните указанную ниже команду. В приведенном ниже примере имя подписки — `ContosoSubscription`.

```azurepowershell-interactive
Select-AzSubscription -SubscriptionName ContosoSubscription
```

<a name="list-search-services"></a>

## <a name="list-services-in-a-subscription"></a>Перечисление служб в подписке

Следующие команды относятся к [**AZ. Resources**](/powershell/module/az.resources)и возвращают сведения о существующих ресурсах и службах, уже подготовленных в вашей подписке. Если вы не уверены, сколько служб поиска уже создано, эти команды возвращают эту информацию, экономя на портале.

Первая команда возвращает все службы поиска.

```azurepowershell-interactive
Get-AzResource -ResourceType Microsoft.Search/searchServices | ft
```

В списке служб верните сведения о конкретном ресурсе.

```azurepowershell-interactive
Get-AzResource -ResourceName <service-name>
```

Результаты должны выглядеть следующим образом.

```
Name              : my-demo-searchapp
ResourceGroupName : demo-westus
ResourceType      : Microsoft.Search/searchServices
Location          : westus
ResourceId        : /subscriptions/<alphanumeric-subscription-ID>/resourceGroups/demo-westus/providers/Microsoft.Search/searchServices/my-demo-searchapp
```

## <a name="import-azsearch"></a>Импорт AZ. Search

Команды из [**AZ. Search**](/powershell/module/az.search) недоступны, пока не будет загружен модуль.

```azurepowershell-interactive
Install-Module -Name Az.Search
```

### <a name="list-all-azsearch-commands"></a>Вывод списка всех команд AZ. Search

На этапе проверки возвращается список команд, предоставленных в модуле.

```azurepowershell-interactive
Get-Command -Module Az.Search
```

Результаты должны выглядеть следующим образом.

```
CommandType     Name                                               Version    Source                                                                
-----------     ----                                               -------    ------                                                                
Cmdlet          Get-AzSearchAdminKeyPair                           0.8.0      Az.Search                                                             
Cmdlet          Get-AzSearchPrivateEndpointConnection              0.8.0      Az.Search                                                             
Cmdlet          Get-AzSearchPrivateLinkResource                    0.8.0      Az.Search                                                             
Cmdlet          Get-AzSearchQueryKey                               0.8.0      Az.Search                                                             
Cmdlet          Get-AzSearchService                                0.8.0      Az.Search                                                             
Cmdlet          Get-AzSearchSharedPrivateLinkResource              0.8.0      Az.Search                                                             
Cmdlet          New-AzSearchAdminKey                               0.8.0      Az.Search                                                             
Cmdlet          New-AzSearchQueryKey                               0.8.0      Az.Search                                                             
Cmdlet          New-AzSearchService                                0.8.0      Az.Search                                                             
Cmdlet          New-AzSearchSharedPrivateLinkResource              0.8.0      Az.Search                                                             
Cmdlet          Remove-AzSearchPrivateEndpointConnection           0.8.0      Az.Search                                                             
Cmdlet          Remove-AzSearchQueryKey                            0.8.0      Az.Search                                                             
Cmdlet          Remove-AzSearchService                             0.8.0      Az.Search                                                             
Cmdlet          Remove-AzSearchSharedPrivateLinkResource           0.8.0      Az.Search                                                             
Cmdlet          Set-AzSearchPrivateEndpointConnection              0.8.0      Az.Search                                                             
Cmdlet          Set-AzSearchService                                0.8.0      Az.Search                                                             
Cmdlet          Set-AzSearchSharedPrivateLinkResource              0.8.0      Az.Search   
```

Если у вас установлена более старая версия пакета, обновите модуль, чтобы получить новейшие функциональные возможности.

```azurepowershell-interactive
Update-Module -Name Az.Search
```

## <a name="get-search-service-information"></a>Получение сведений о службе поиска

После команды **AZ. Search** импортируется, и вы узнаете о группе ресурсов, содержащей службу поиска, выполните командлет [Get-азсеарчсервице](/powershell/module/az.search/get-azsearchservice) , чтобы получить определение службы, включая имя, регион, уровень, реплику и число разделов.

```azurepowershell-interactive
Get-AzSearchService -ResourceGroupName <resource-group-name>
```

Результаты должны выглядеть следующим образом.

```
Name              : my-demo-searchapp
ResourceGroupName : demo-westus
ResourceType      : Microsoft.Search/searchServices
Location          : West US
Sku               : Standard
ReplicaCount      : 1
PartitionCount    : 1
HostingMode       : Default
ResourceId        : /subscriptions/<alphanumeric-subscription-ID>/resourceGroups/demo-westus/providers/Microsoft.Search/searchServices/my-demo-searchapp
```

## <a name="create-or-delete-a-service"></a>Создание или удаление службы

[**New-азсеарчсервице**](/powershell/module/az.search/new-azsearchservice) используется для [создания новой службы поиска](search-create-service-portal.md).

```azurepowershell-interactive
New-AzSearchService -ResourceGroupName <resource-group-name> -Name <search-service-name> -Sku "Standard" -Location "West US" -PartitionCount 3 -ReplicaCount 3 -HostingMode Default
``` 
Результаты должны выглядеть следующим образом.

```
ResourceGroupName : demo-westus
Name              : my-demo-searchapp
Id                : /subscriptions/<alphanumeric-subscription-ID>/demo-westus/providers/Microsoft.Search/searchServices/my-demo-searchapp
Location          : West US
Sku               : Standard
ReplicaCount      : 3
PartitionCount    : 3
HostingMode       : Default
Tags
```     

### <a name="create-a-service-with-ip-rules"></a>Создание службы с правилами IP-адресов

В зависимости от требований безопасности может потребоваться создать службу поиска с [настроенным брандмауэром IP](service-configure-firewall.md). Для этого сначала определите правила IP-адресов, а затем передайте их в `IPRuleList` параметр, как показано ниже.

```azurepowershell-interactive
$ipRules = @([pscustomobject]@{Value="55.5.63.73"},
        [pscustomobject]@{Value="52.228.215.197"},
        [pscustomobject]@{Value="101.37.221.205"})

 New-AzSearchService -ResourceGroupName <resource-group-name> `
                      -Name <search-service-name> `
                      -Sku Standard `
                      -Location "West US" `
                      -PartitionCount 3 -ReplicaCount 3 `
                      -HostingMode Default `
                      -IPRuleList $ipRules
```

### <a name="create-a-service-with-a-system-assigned-managed-identity"></a>Создание службы с управляемым удостоверением, назначенным системой

В некоторых случаях, например при [использовании управляемого удостоверения для подключения к источнику данных](search-howto-managed-identities-storage.md), необходимо включить [управляемое удостоверение, назначенное системой](../active-directory/managed-identities-azure-resources/overview.md). Это можно сделать, добавив `-IdentityType SystemAssigned` к команде.

```azurepowershell-interactive
New-AzSearchService -ResourceGroupName <resource-group-name> `
                      -Name <search-service-name> `
                      -Sku Standard `
                      -Location "West US" `
                      -PartitionCount 3 -ReplicaCount 3 `
                      -HostingMode Default `
                      -IdentityType SystemAssigned
```

## <a name="create-a-service-with-a-private-endpoint"></a>Создание службы с закрытой конечной точкой

[Частные конечные точки](../private-link/private-endpoint-overview.md) для Azure когнитивный Поиск позволяют клиенту в виртуальной сети безопасно получать доступ к данным в индексе поиска по [закрытой ссылке](../private-link/private-link-overview.md). Частная конечная точка использует IP-адрес из [адресного пространства виртуальной сети](../virtual-network/private-ip-addresses.md) для службы поиска. Сетевой трафик между клиентом и службой поиска проходит по виртуальной сети и частной связи в магистральной сети Майкрософт, что устраняет уязвимость в общедоступном Интернете. Дополнительные сведения см. в документации по [созданию частной конечной точки для Azure когнитивный Поиск](service-create-private-endpoint.md)

В следующем примере показано, как создать службу поиска с закрытой конечной точкой.

Сначала разверните службу поиска со `PublicNetworkAccess` значением `Disabled` .

```azurepowershell-interactive
$searchService = New-AzSearchService `
    -ResourceGroupName <resource-group-name> `
    -Name <search-service-name> `
    -Sku Standard `
    -Location "West US" `
    -PartitionCount 1 -ReplicaCount 1 `
    -HostingMode Default `
    -PublicNetworkAccess Disabled
```

Затем создайте виртуальную сеть, подключение к частной сети и закрытую конечную точку.

```azurepowershell-interactive
# Create the subnet
$subnetConfig = New-AzVirtualNetworkSubnetConfig `
    -Name <subnet-name> `
    -AddressPrefix 10.1.0.0/24 `
    -PrivateEndpointNetworkPolicies Disabled 

# Create the virtual network
$virtualNetwork = New-AzVirtualNetwork `
    -ResourceGroupName <resource-group-name> `
    -Location "West US" `
    -Name <virtual-network-name> `
    -AddressPrefix 10.1.0.0/16 `
    -Subnet $subnetConfig

# Create the private network connection
$privateLinkConnection = New-AzPrivateLinkServiceConnection `
    -Name <private-link-name> `
    -PrivateLinkServiceId $searchService.Id `
    -GroupId searchService

# Create the private endpoint
$privateEndpoint = New-AzPrivateEndpoint `
    -Name <private-endpoint-name> `
    -ResourceGroupName <resource-group-name> `
    -Location "West US" `
    -Subnet $virtualNetwork.subnets[0] `
    -PrivateLinkServiceConnection $privateLinkConnection
```

Наконец, создайте частную зону DNS. 

```azurepowershell-interactive
## Create private dns zone
$zone = New-AzPrivateDnsZone `
    -ResourceGroupName <resource-group-name> `
    -Name "privatelink.search.windows.net"

## Create dns network link
$link = New-AzPrivateDnsVirtualNetworkLink `
    -ResourceGroupName <resource-group-name> `
    -ZoneName "privatelink.search.windows.net" `
    -Name "myLink" `
    -VirtualNetworkId $virtualNetwork.Id

## Create DNS configuration 
$config = New-AzPrivateDnsZoneConfig `
    -Name "privatelink.search.windows.net" `
    -PrivateDnsZoneId $zone.ResourceId

## Create DNS zone group
New-AzPrivateDnsZoneGroup `
    -ResourceGroupName <resource-group-name> `
    -PrivateEndpointName <private-endpoint-name> `
    -Name 'myZoneGroup' `
    -PrivateDnsZoneConfig $config
```

Дополнительные сведения о создании частных конечных точек в PowerShell см. в этом [кратком руководстве по этой частной ссылке](../private-link/create-private-endpoint-powershell.md) .

### <a name="manage-private-endpoint-connections"></a>Управление подключениями к частным конечным точкам

Кроме создания подключения к частной конечной точке, можно также `Get` `Set` и `Remove` подключение.

[Get-азсеарчприватиндпоинтконнектион](/powershell/module/az.search/Get-AzSearchPrivateEndpointConnection) используется для получения подключения к частной конечной точке и просмотра его состояния.

```azurepowershell-interactive
Get-AzSearchPrivateEndpointConnection -ResourceGroupName <resource-group-name> -ServiceName <search-service-name>
```

[Set-азсеарчприватиндпоинтконнектион](/powershell/module/az.search/Set-AzSearchPrivateEndpointConnection) используется для обновления соединения. В следующем примере задается отклонение подключения к частной конечной точке:

```azurepowershell-interactive
Set-AzSearchPrivateEndpointConnection -ResourceGroupName <resource-group-name> -ServiceName <search-service-name> -Name <pe-connection-name> -Status Rejected  -Description "Rejected"
```

[Remove-азсеарчприватиндпоинтконнектион](/powershell/module/az.search/Remove-AzSearchPrivateEndpointConnection) используется для удаления подключения к частной конечной точке.

```azurepowershell-interactive
 Remove-AzSearchPrivateEndpointConnection -ResourceGroupName <resource-group-name> -ServiceName <search-service-name> -Name <pe-connection-name>
```

## <a name="regenerate-admin-keys"></a>Повторное создание ключей администратора

[**New-азсеарчадминкэй**](/powershell/module/az.search/new-azsearchadminkey) используется для отката [ключей API](search-security-api-keys.md)администратора. Для каждой службы создаются два ключа администратора для доступа с проверкой подлинности. Для каждого запроса требуются ключи. Оба ключа администратора функционально эквивалентны, они предоставляют полный доступ для записи службе поиска с возможностью получения любых сведений, а также создания и удаления любого объекта. Существует два ключа, которые можно использовать при замене другого. 

Его можно создать только по одному, указанному в качестве `primary` `secondary` ключа или. При использовании непрерывной службы не забудьте обновить весь код клиента для использования вторичного ключа при последовательном переходе по первичному ключу. Старайтесь не менять ключи, пока операции находятся в полете.

Как вы можете ожидать, при повторном создании ключей без обновления клиентского кода запросы, использующие старый ключ, завершатся ошибкой. Повторное создание всех новых ключей не приводит к окончательной блокировке вашей службы, и вы по-прежнему можете обращаться к службе через портал. После повторного создания первичного и вторичного ключей можно обновить код клиента, чтобы использовать новые ключи, и операции будут возобновлены соответствующим образом.

Значения ключей API создаются службой. Вы не можете предоставить настраиваемый ключ для Azure Когнитивный поиск. Точно так же не существует определяемого пользователем имени для ключей API администратора. Ссылки на ключ являются фиксированными строками: `primary` или `secondary` . 

```azurepowershell-interactive
New-AzSearchAdminKey -ResourceGroupName <resource-group-name> -ServiceName <search-service-name> -KeyKind Primary
```

Результаты должны выглядеть следующим образом. Возвращаются оба ключа, несмотря на то, что вы изменяете их только один раз.

```
Primary                    Secondary
-------                    ---------
<alphanumeric-guid>        <alphanumeric-guid>  
```

## <a name="create-or-delete-query-keys"></a>Создание или удаление ключей запроса

[**New-азсеарчкуерикэй**](/powershell/module/az.search/new-azsearchquerykey) используется для создания [ключей API](search-security-api-keys.md) запросов для доступа только для чтения из клиентских приложений к индексу Azure когнитивный Поиск. Ключи запроса используются для проверки подлинности определенного индекса с целью получения результатов поиска. Ключи запроса не предоставляют доступ только для чтения к другим элементам службы, таким как индекс, источник данных или индексатор.

Вы не можете предоставить ключ для использования Когнитивный поиск Azure. Ключи API создаются службой.

```azurepowershell-interactive
New-AzSearchQueryKey -ResourceGroupName <resource-group-name> -ServiceName <search-service-name> -Name <query-key-name> 
```

## <a name="scale-replicas-and-partitions"></a>Масштабирование реплик и секций

[**Set-азсеарчсервице**](/powershell/module/az.search/set-azsearchservice) используется для [увеличения или уменьшения количества реплик и секций](search-capacity-planning.md) , чтобы перенастроить оплачиваемые ресурсы в службе. При увеличении количества реплик или секций добавляется к счету, у которого есть фиксированная и переменная плата. При наличии временной потребности в дополнительной вычислительной мощности можно увеличить реплики и секции для обработки рабочей нагрузки. В области мониторинг на странице Обзор портала находятся плитки о задержке запросов, запросах в секунду и регулированиях, указывающих, достаточно ли текущих мощностей.

Добавление или удаление перебора может занять некоторое время. Корректировки производительности выполняются в фоновом режиме, что позволяет продолжать работу существующих рабочих нагрузок. Дополнительная емкость используется для входящих запросов, как только она будет готова, без необходимости дополнительной настройки. 

Удаление емкости может нарушить работу. Остановка всех заданий индексирования и индексирования перед уменьшением емкости рекомендуется, чтобы избежать отброшенных запросов. Если это нецелесообразно, можно потребовать более постепенного снижения емкости по одной реплике и секции, пока не будут достигнуты новые целевые уровни.

После отправки команды нет возможности завершить ее с самого посредине. Перед изменением счетчиков необходимо будет дождаться завершения команды.

```azurepowershell-interactive
Set-AzSearchService -ResourceGroupName <resource-group-name> -Name <search-service-name> -PartitionCount 6 -ReplicaCount 6
```

Результаты должны выглядеть следующим образом.

```
ResourceGroupName : demo-westus
Name              : my-demo-searchapp
Location          : West US
Sku               : Standard
ReplicaCount      : 6
PartitionCount    : 6
HostingMode       : Default
Id                : /subscriptions/65a1016d-0f67-45d2-b838-b8f373d6d52e/resourceGroups/demo-westus/providers/Microsoft.Search/searchServices/my-demo-searchapp
```

## <a name="create-a-shared-private-link-resource"></a>Создание общего ресурса частной ссылки

Частные конечные точки защищенных ресурсов, созданные с помощью Azure Когнитивный поиск API, называются *общими ресурсами частной ссылки*. Это связано с тем, что вы используете общий доступ к ресурсу, например к учетной записи хранения, интегрированной со [службой частной связи Azure](https://azure.microsoft.com/services/private-link/).

Если вы используете индексатор для индексирования данных в Когнитивный поиск Azure, а источник данных находится в частной сети, можно создать исходящее [Подключение к частной конечной точке](../private-link/private-endpoint-overview.md) для достижения данных.

Полный список ресурсов Azure, для которых можно создать исходящие частные конечные точки из Когнитивный поиск Azure, можно найти [здесь](search-indexer-howto-access-private.md#shared-private-link-resources-management-apis) вместе со связанными значениями **идентификаторов групп** .

[New-азсеарчшаредпривателинкресаурце](/powershell/module/az.search/New-AzSearchSharedPrivateLinkResource) используется для создания общего ресурса частной ссылки. Помните, что перед выполнением этой команды может потребоваться определенная конфигурация для источника данных.

```azurepowershell-interactive
New-AzSearchSharedPrivateLinkResource -ResourceGroupName <resource-group-name> -ServiceName <search-service-name> -Name <spl-name> -PrivateLinkResourceId /subscriptions/<alphanumeric-subscription-ID>/resourceGroups/<resource-group-name>/providers/Microsoft.Storage/storageAccounts/myBlobStorage -GroupId <group-id> -RequestMessage "Please approve" 
```

[Get-азсеарчшаредпривателинкресаурце](/powershell/module/az.search/Get-AzSearchSharedPrivateLinkResource) позволяет получить общие ресурсы частной ссылки и просмотреть их состояние.

```azurepowershell-interactive
Get-AzSearchSharedPrivateLinkResource -ResourceGroupName <resource-group-name> -ServiceName <search-service-name> -Name <spl-name>
```

Прежде чем можно будет использовать это подключение, необходимо одобрить его с помощью следующей команды.

```azurepowershell-interactive
Approve-AzPrivateEndpointConnection `
    -Name <spl-name> `
    -ServiceName <search-service-name> `
    -ResourceGroupName <resource-group-name> `
    -Description = "Approved"
```

[Remove-азсеарчшаредпривателинкресаурце](/powershell/module/az.search/Remove-AzSearchSharedPrivateLinkResource) используется для удаления общего ресурса частной ссылки.

```azurepowershell-interactive
$job = Remove-AzSearchSharedPrivateLinkResource -ResourceGroupName <resource-group-name> -ServiceName <search-service-name> -Name <spl-name> -Force -AsJob

$job | Get-Job
```

Дополнительные сведения о настройке общих ресурсов частной связи см. в документации по [созданию подключений индексатора через закрытую конечную точку](search-indexer-howto-access-private.md).

## <a name="next-steps"></a>Дальнейшие действия

Создайте [индекс](search-what-is-an-index.md), [запросите индекс](search-query-overview.md) с помощью портала, API-интерфейсов и пакета SDK для .NET.

* [Создание индекса службы "Когнитивный поиск Azure" на портале Azure](search-get-started-portal.md)
* [Настройка индексатора для загрузки данных из других служб](search-indexer-overview.md)
* [Запрос индекса Когнитивный поиск Azure с помощью обозревателя поиска в портал Azure](search-explorer.md)
* [Использование Когнитивный поиск Azure в .NET](search-howto-dotnet-sdk.md)
