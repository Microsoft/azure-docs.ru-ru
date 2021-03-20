---
title: Использование PowerShell для управления диспетчером трафика в Azure
description: По этому пути обучения приступайте к использованию Azure PowerShell для диспетчера трафика.
services: traffic-manager
documentationcenter: na
author: duongau
ms.service: traffic-manager
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/16/2017
ms.author: duau
ms.openlocfilehash: 21076fe23301c189d9987f78706cc81691ce7a4f
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "89400574"
---
# <a name="using-powershell-to-manage-traffic-manager"></a>Использование PowerShell для управления диспетчером трафика

Azure Resource Manager представляет собой предпочтительный интерфейс управления для служб в Azure. Профилями диспетчера трафика Azure можно управлять с помощью интерфейсов API и инструментов на основе Azure Resource Manager.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="resource-model"></a>Модель ресурсов

Для настройки диспетчера трафика используется набор параметров, которые называются профилем. Профиль содержит параметры DNS, параметры маршрутизации трафика, параметры отслеживания конечных точек, а также список конечных точек службы, на которые выполняется маршрутизация трафика.

Каждый профиль диспетчера трафика представлен ресурсом типа TrafficManagerProfiles. На уровне API REST каждый профиль имеет следующий URI:

`https://management.azure.com/subscriptions/{subscription-id}/resourceGroups/{resource-group-name}/providers/Microsoft.Network/trafficManagerProfiles/{profile-name}?api-version={api-version}`

## <a name="setting-up-azure-powershell"></a>Настройка Azure PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

В данных инструкциях используется Microsoft Azure PowerShell. В следующей статье объясняется, как установить и настроить Azure PowerShell:

* [Как установить и настроить Azure PowerShell](/powershell/azure/)

Для приведенных в этой статье примеров предполагается, что у вас уже есть группа ресурсов. Чтобы создать группу ресурсов, выполните следующую команду:

```powershell
New-AzResourceGroup -Name MyRG -Location "West US"
```

> [!NOTE]
> Для Azure Resource Manager необходимо, чтобы все группы ресурсов имели расположение. Это расположение используется в качестве расположения по умолчанию для ресурсов, созданных в этой группе ресурсов. Но так как ресурсы профиля диспетчера трафика являются глобальными, а не региональными, выбор расположения группы ресурсов никак не повлияет на работу диспетчера трафика Azure.

## <a name="create-a-traffic-manager-profile"></a>Создание профиля диспетчера трафика

Чтобы создать профиль диспетчера трафика, используйте командлет `New-AzTrafficManagerProfile`:

```powershell
$TmProfile = New-AzTrafficManagerProfile -Name MyProfile -ResourceGroupName MyRG -TrafficRoutingMethod Performance -RelativeDnsName contoso -Ttl 30 -MonitorProtocol HTTP -MonitorPort 80 -MonitorPath "/"
```

В следующей таблице описаны параметры.

| Параметр | Описание |
| --- | --- |
| Имя |Имя ресурса профиля диспетчера трафика. У профилей в одной группе ресурсов должны быть уникальные имена. Это имя отличается от имени DNS, которое используется в запросах DNS. |
| ResourceGroupName |Имя группы ресурсов, которая содержит ресурс профиля. |
| TrafficRoutingMethod |Задает метод маршрутизации трафика, который определяет конечную точку, возвращаемую в ответ на запрос DNS. Возможные значения: Performance (производительность), Weighted (взвешенный) и Priority (приоритетный). |
| RelativeDnsName |Задает часть DNS-имени с именем узла, которое предоставляется этим профилем диспетчера трафика. Это значение объединяется с DNS-именем домена, с помощью которого диспетчер трафика Azure формирует полное доменное имя профиля. Например, если задать значение contoso, полное доменное имя будет contoso.trafficmanager.net. |
| Срок жизни |Задает срок жизни запроса DNS в секундах. Это значение срока жизни дает локальным сопоставителям DNS и DNS-клиентам понять, в течение какого срока нужно хранить в кэше ответы DNS для этого профиля диспетчера трафика. |
| MonitorProtocol |Задает протокол, который следует использовать для контроля работоспособности конечных точек. Допустимые значения: HTTP и HTTPS. |
| MonitorPort |Задает порт TCP, который следует использовать для контроля работоспособности конечных точек. |
| MonitorPath |Задает путь относительно доменного имени конечной точки, который используется для проверки ее работоспособности. |

Командлет создает профиль диспетчера трафика в Azure и возвращает соответствующий объект профиля в PowerShell. На этом этапе профиль не содержит конечные точки. Дополнительные сведения о добавлении конечных точек в профиль диспетчера трафика см. в разделе о добавлении конечных точек диспетчера трафика.

## <a name="get-a-traffic-manager-profile"></a>Получение профиля диспетчера трафика

Получить существующий объект профиля диспетчера трафика можно с помощью командлета `Get-AzTrafficManagerProfle`:

```powershell
$TmProfile = Get-AzTrafficManagerProfile -Name MyProfile -ResourceGroupName MyRG
```

Этот командлет возвращает профиль объекта трафика.

## <a name="update-a-traffic-manager-profile"></a>Изменение профиля диспетчера трафика

Изменение профилей диспетчера трафика выполняется в три этапа.

1. Получите профиль с помощью командлета `Get-AzTrafficManagerProfile` или используйте профиль, возвращенный командлетом `New-AzTrafficManagerProfile`.
2. Измените профиль. Можно добавить и удалить конечные точки или изменить параметры профиля или конечной точки. Эти изменения выполняются в автономном режиме. Изменяется только локальный объект в памяти, представляющий профиль.
3. Зафиксируйте изменения с помощью командлета `Set-AzTrafficManagerProfile`.

Можно изменить все свойства профиля, кроме RelativeDnsName. Чтобы изменить свойство RelativeDnsName, необходимо удалить профиль и создать новый профиль с новым именем.

В следующем примере показано, как изменить в профиле срок жизни.

```powershell
$TmProfile = Get-AzTrafficManagerProfile -Name MyProfile -ResourceGroupName MyRG
$TmProfile.Ttl = 300
Set-AzTrafficManagerProfile -TrafficManagerProfile $TmProfile
```

Существует три типа конечных точек диспетчера трафика.

1. **Конечные точки Azure** — это службы, размещенные в Azure.
2. **Внешние конечные точки** — службы, размещенные за пределами Azure.
3. **Вложенные конечные точки** используются для создания вложенных иерархий профилей диспетчера трафика. Вложенные конечные точки позволяют создавать расширенные конфигурации маршрутизации трафика для сложных приложений.

Конечные точки всех трех типов можно добавлять двумя способами.

1. С помощью описанных выше трех шагов. Этот метод удобен тем, что позволяет в ходе одного обновления внести несколько изменений в настройки конечной точки.
2. С помощью командлета New-AzTrafficManagerEndpoint. Он добавляет конечную точку в существующий профиль диспетчера трафика за одну операцию.

## <a name="adding-azure-endpoints"></a>Добавление конечных точек Azure

Конечные точки Azure ссылаются на службы, размещенные в Azure. Поддерживаются два типа конечных точек Azure.

1. Служба приложений Azure
2. Ресурсы Azure PublicIpAddress, которые могут подключаться к подсистеме балансировки нагрузки или к сетевому адаптеру виртуальной машины. Для использования в диспетчере трафика конечной точке publicIpAddress должно быть назначено DNS-имя.

В каждом случае:

* Для указания службы используется параметр targetResourceId командлета `Add-AzTrafficManagerEndpointConfig` или `New-AzTrafficManagerEndpoint`.
* Параметры Target и EndpointLocation определяются с помощью параметра TargetResourceId.
* Указывать Weight необязательно. Параметр Weight используется только в том случае, если профиль настроен для использования взвешенного (weighted) метода маршрутизации трафика. В остальных случаях этот параметр игнорируется. Если этот параметр указывается, его значение должно быть числом в диапазоне от 1 до 1000. Значение по умолчанию — 1.
* Указывать Priority необязательно. Параметр Priority используется только в том случае, если профиль настроен для использования приоритетного (priority) метода маршрутизации. В остальных случаях этот параметр игнорируется. Допускаются значения от 1 до 1000 (чем меньше число, тем выше приоритет). Если приоритет указан для одной конечной точки, он должен указываться и для всех остальных конечных точек. Если значения не указаны, конечным точкам в порядке очереди присваиваются значения по умолчанию: 1 и т. д.

### <a name="example-1-adding-app-service-endpoints-using-add-aztrafficmanagerendpointconfig"></a>Пример 1. Добавление конечных точек службы приложений с помощью `Add-AzTrafficManagerEndpointConfig`

В этом примере мы создадим профиль диспетчера трафика и добавим две конечные точки службы приложений с помощью командлета `Add-AzTrafficManagerEndpointConfig`.

```powershell
$TmProfile = New-AzTrafficManagerProfile -Name myprofile -ResourceGroupName MyRG -TrafficRoutingMethod Performance -RelativeDnsName myapp -Ttl 30 -MonitorProtocol HTTP -MonitorPort 80 -MonitorPath "/"
$webapp1 = Get-AzWebApp -Name webapp1
Add-AzTrafficManagerEndpointConfig -EndpointName webapp1ep -TrafficManagerProfile $TmProfile -Type AzureEndpoints -TargetResourceId $webapp1.Id -EndpointStatus Enabled
$webapp2 = Get-AzWebApp -Name webapp2
Add-AzTrafficManagerEndpointConfig -EndpointName webapp2ep -TrafficManagerProfile $TmProfile -Type AzureEndpoints -TargetResourceId $webapp2.Id -EndpointStatus Enabled
Set-AzTrafficManagerProfile -TrafficManagerProfile $TmProfile
```
### <a name="example-2-adding-a-publicipaddress-endpoint-using-new-aztrafficmanagerendpoint"></a>Пример 2. Добавление конечной точки publicIpAddress с помощью командлета `New-AzTrafficManagerEndpoint`

В этом примере мы добавляем в профиль диспетчера трафика ресурс с общедоступным IP-адресом. Общедоступный IP-адрес должен включать DNS-имя и может быть привязан к сетевому адаптеру виртуальной машины или к балансировщику нагрузки.

```powershell
$ip = Get-AzPublicIpAddress -Name MyPublicIP -ResourceGroupName MyRG
New-AzTrafficManagerEndpoint -Name MyIpEndpoint -ProfileName MyProfile -ResourceGroupName MyRG -Type AzureEndpoints -TargetResourceId $ip.Id -EndpointStatus Enabled
```

## <a name="adding-external-endpoints"></a>Добавление внешних конечных точек

Диспетчер трафика использует внешние конечные точки для передачи трафика в службы, размещенные за пределами Azure. Как и конечные точки Azure, внешние конечные точки можно добавить с помощью командлета `Add-AzTrafficManagerEndpointConfig`, за которым следует командлет `Set-AzTrafficManagerProfile` или `New-AzTrafficManagerEndpoint`.

При добавлении внешних конечных точек:

* Доменное имя конечной точки должно быть указано с помощью параметра Target.
* EndpointLocation является обязательным параметром, если используется метод маршрутизации трафика Performance. В противном случае этот параметр является необязательным. Его значением должно быть [допустимое имя региона Azure](https://azure.microsoft.com/regions/).
* Параметры Weight и Priority являются необязательными.

### <a name="example-1-adding-external-endpoints-using-add-aztrafficmanagerendpointconfig-and-set-aztrafficmanagerprofile"></a>Пример 1. Добавление внешних конечных точек с помощью командлетов `Add-AzTrafficManagerEndpointConfig` и `Set-AzTrafficManagerProfile`

В этом примере мы создаем профиль диспетчера трафика, добавляем в него две внешние конечные точки и фиксируем изменения.

```powershell
$TmProfile = New-AzTrafficManagerProfile -Name myprofile -ResourceGroupName MyRG -TrafficRoutingMethod Performance -RelativeDnsName myapp -Ttl 30 -MonitorProtocol HTTP -MonitorPort 80 -MonitorPath "/"
Add-AzTrafficManagerEndpointConfig -EndpointName eu-endpoint -TrafficManagerProfile $TmProfile -Type ExternalEndpoints -Target app-eu.contoso.com -EndpointLocation "North Europe" -EndpointStatus Enabled
Add-AzTrafficManagerEndpointConfig -EndpointName us-endpoint -TrafficManagerProfile $TmProfile -Type ExternalEndpoints -Target app-us.contoso.com -EndpointLocation "Central US" -EndpointStatus Enabled
Set-AzTrafficManagerProfile -TrafficManagerProfile $TmProfile
```

### <a name="example-2-adding-external-endpoints-using-new-aztrafficmanagerendpoint"></a>Пример 2. Добавление внешних конечных точек с помощью командлета `New-AzTrafficManagerEndpoint`

В этом примере мы добавляем внешнюю конечную точку в существующий профиль. Профиль указывается с помощью имени профиля и имени группы ресурсов.

```powershell
New-AzTrafficManagerEndpoint -Name eu-endpoint -ProfileName MyProfile -ResourceGroupName MyRG -Type ExternalEndpoints -Target app-eu.contoso.com -EndpointStatus Enabled
```

## <a name="adding-nested-endpoints"></a>Добавление "вложенных" конечных точек

Каждый профиль диспетчера трафика определяет один метод маршрутизации трафика. Но бывают случаи, когда требуется более сложная маршрутизация трафика, чем та, которую может обеспечить отдельный профиль диспетчера трафика. Чтобы воспользоваться преимуществами нескольких методов маршрутизации трафика, можно выполнить вложение профилей диспетчера трафика. Вложенные профили позволяют переопределить стандартное поведение диспетчера трафика, реализуя поддержку крупных и сложных развертываний приложений. Более подробные примеры см. в статье [Вложенные профили диспетчера трафика](traffic-manager-nested-profiles.md).

Вложенные конечные точки настраиваются в родительском профиле по типу конечной точки "NestedEndpoints". При указании вложенных конечных точек:

* Конечная точка должна быть указана с помощью параметра targetResourceId.
* EndpointLocation является обязательным параметром, если используется метод маршрутизации трафика Performance. В противном случае этот параметр является необязательным. Его значением должно быть [допустимое имя региона Azure](https://azure.microsoft.com/regions/).
* Параметры Weight и Priority для конечных точек Azure являются необязательными.
* Параметр MinChildEndpoints является необязательным. Значение по умолчанию — 1. Если количество доступных конечных точек опускается ниже этого значения, родительский профиль будет рассматривать дочерний профиль как профиль более низкого уровня, направляя трафик к другим конечным точкам родительского профиля.

### <a name="example-1-adding-nested-endpoints-using-add-aztrafficmanagerendpointconfig-and-set-aztrafficmanagerprofile"></a>Пример 1. Добавление вложенных конечных точек с помощью командлетов `Add-AzTrafficManagerEndpointConfig` и `Set-AzTrafficManagerProfile`

В этом примере мы создаем новые дочерний и родительский профили диспетчера трафика, добавляем дочерний профиль как вложенный в родительский профиль и фиксируем изменения.

```powershell
$child = New-AzTrafficManagerProfile -Name child -ResourceGroupName MyRG -TrafficRoutingMethod Priority -RelativeDnsName child -Ttl 30 -MonitorProtocol HTTP -MonitorPort 80 -MonitorPath "/"
$parent = New-AzTrafficManagerProfile -Name parent -ResourceGroupName MyRG -TrafficRoutingMethod Performance -RelativeDnsName parent -Ttl 30 -MonitorProtocol HTTP -MonitorPort 80 -MonitorPath "/"
Add-AzTrafficManagerEndpointConfig -EndpointName child-endpoint -TrafficManagerProfile $parent -Type NestedEndpoints -TargetResourceId $child.Id -EndpointStatus Enabled -EndpointLocation "North Europe" -MinChildEndpoints 2
Set-AzTrafficManagerProfile -TrafficManagerProfile $parent
```

Для краткости мы не добавляли другие конечные точки в родительский и дочерний профили.

### <a name="example-2-adding-nested-endpoints-using-new-aztrafficmanagerendpoint"></a>Пример 2. Добавление вложенных конечных точек с помощью командлета `New-AzTrafficManagerEndpoint`

В этом примере мы добавляем существующий дочерний профиль как вложенную конечную точку в существующий родительский профиль. Профиль указывается с помощью имени профиля и имени группы ресурсов.

```powershell
$child = Get-AzTrafficManagerEndpoint -Name child -ResourceGroupName MyRG
New-AzTrafficManagerEndpoint -Name child-endpoint -ProfileName parent -ResourceGroupName MyRG -Type NestedEndpoints -TargetResourceId $child.Id -EndpointStatus Enabled -EndpointLocation "North Europe" -MinChildEndpoints 2
```

## <a name="adding-endpoints-from-another-subscription"></a>Добавление конечных точек из другой подписки

Диспетчер трафика может работать с конечными точками из нескольких подписок. Вам нужно переключиться на подписку с конечной точкой, которую вы хотите добавить для получения необходимых входных данных для диспетчера трафика. Затем необходимо переключиться на подписки с помощью профиля диспетчера трафика и добавить в нее конечную точку. В примере ниже показано, как это сделать с помощью общедоступного IP-адреса.

```powershell
Set-AzContext -SubscriptionId $EndpointSubscription
$ip = Get-AzPublicIpAddress -Name $IpAddressName -ResourceGroupName $EndpointRG

Set-AzContext -SubscriptionId $trafficmanagerSubscription
New-AzTrafficManagerEndpoint -Name $EndpointName -ProfileName $ProfileName -ResourceGroupName $TrafficManagerRG -Type AzureEndpoints -TargetResourceId $ip.Id -EndpointStatus Enabled
```

## <a name="update-a-traffic-manager-endpoint"></a>Обновление конечной точки диспетчера трафика

Есть два способа обновления существующей конечной точки диспетчера трафика.

1. Получить профиль диспетчера трафика с помощью командлета `Get-AzTrafficManagerProfile`, обновить свойства конечной точки в профиле и зафиксировать изменения с помощью `Set-AzTrafficManagerProfile`. Этот метод удобен тем, что позволяет в ходе одной операции обновления изменить настройки нескольких конечных точек.
2. Получить конечную точку диспетчера трафика с помощью командлета `Get-AzTrafficManagerEndpoint`, обновить свойства конечной точки и зафиксировать изменения с помощью командлета `Set-AzTrafficManagerEndpoint`. Этот метод проще, так как он не требует индексации массива конечных точек в профиле.

### <a name="example-1-updating-endpoints-using-get-aztrafficmanagerprofile-and-set-aztrafficmanagerprofile"></a>Пример 1. Обновление конечных точек с помощью командлетов `Get-AzTrafficManagerProfile` и `Set-AzTrafficManagerProfile`

В этом примере мы изменим приоритет двух конечных точек в существующем профиле.

```powershell
$TmProfile = Get-AzTrafficManagerProfile -Name myprofile -ResourceGroupName MyRG
$TmProfile.Endpoints[0].Priority = 2
$TmProfile.Endpoints[1].Priority = 1
Set-AzTrafficManagerProfile -TrafficManagerProfile $TmProfile
```

### <a name="example-2-updating-an-endpoint-using-get-aztrafficmanagerendpoint-and-set-aztrafficmanagerendpoint"></a>Пример 2. Обновление конечной точки с помощью командлетов `Get-AzTrafficManagerEndpoint` и `Set-AzTrafficManagerEndpoint`

В этом примере мы изменим вес одной конечной точки в существующем профиле.

```powershell
$endpoint = Get-AzTrafficManagerEndpoint -Name myendpoint -ProfileName myprofile -ResourceGroupName MyRG -Type ExternalEndpoints
$endpoint.Weight = 20
Set-AzTrafficManagerEndpoint -TrafficManagerEndpoint $endpoint
```

## <a name="enabling-and-disabling-endpoints-and-profiles"></a>Включение и отключение конечных точек и профилей

Диспетчер трафика позволяет включать и отключать как отдельные конечные точки, так и целые профили.
Для этого нужно получить, обновить и зафиксировать ресурсы конечной точки или профиля. Для упрощения этих стандартных операций существуют специальные командлеты.

### <a name="example-1-enabling-and-disabling-a-traffic-manager-profile"></a>Пример 1. Включение и отключение профиля диспетчера трафика

Для включения профиля диспетчера трафика используйте командлет `Enable-AzTrafficManagerProfile`. Профиль можно указать с помощью объекта профиля. Объект профиля можно передать по конвейеру или с помощью параметра -TrafficManagerProfile. В этом примере мы укажем профиль с помощью имени профиля и имени группы ресурсов.

```powershell
Enable-AzTrafficManagerProfile -Name MyProfile -ResourceGroupName MyResourceGroup
```

Отключение профиля диспетчера трафика:

```powershell
Disable-AzTrafficManagerProfile -Name MyProfile -ResourceGroupName MyResourceGroup
```

Командлет Disable-AzTrafficManagerProfile запрашивает подтверждение. Этот запрос можно скрыть с помощью параметра -Force.

### <a name="example-2-enabling-and-disabling-a-traffic-manager-endpoint"></a>Пример 2. Включение и отключение конечной точки диспетчера трафика

Для включения конечной точки диспетчера трафика используйте командлет `Enable-AzTrafficManagerEndpoint`. Задать конечную точку можно двумя способами:

1. С помощью объекта TrafficManagerEndpoint, передаваемого через конвейер, или с помощью параметра -TrafficManagerEndpoint.
2. С помощью имени конечной точки, ее типа, имени профиля и имени группы ресурсов:

```powershell
Enable-AzTrafficManagerEndpoint -Name MyEndpoint -Type AzureEndpoints -ProfileName MyProfile -ResourceGroupName MyRG
```

Аналогичным образом можно отключить конечную точку диспетчера трафика:

```powershell
Disable-AzTrafficManagerEndpoint -Name MyEndpoint -Type AzureEndpoints -ProfileName MyProfile -ResourceGroupName MyRG -Force
```

Как и командлет `Disable-AzTrafficManagerProfile`, `Disable-AzTrafficManagerEndpoint` запрашивает подтверждение. Этот запрос можно скрыть с помощью параметра -Force.

## <a name="delete-a-traffic-manager-endpoint"></a>Удаление конечной точки диспетчера трафика

Для удаления отдельных конечных точек используйте командлет `Remove-AzTrafficManagerEndpoint`:

```powershell
Remove-AzTrafficManagerEndpoint -Name MyEndpoint -Type AzureEndpoints -ProfileName MyProfile -ResourceGroupName MyRG
```

Этот командлет запрашивает подтверждение. Этот запрос можно скрыть с помощью параметра -Force.

## <a name="delete-a-traffic-manager-profile"></a>Удаление профиля диспетчера трафика

Чтобы удалить профиль диспетчера трафика, используйте командлет `Remove-AzTrafficManagerProfile`, указав имя профиля и имя группы ресурсов:

```powershell
Remove-AzTrafficManagerProfile -Name MyProfile -ResourceGroupName MyRG [-Force]
```

Этот командлет запрашивает подтверждение. Этот запрос можно скрыть с помощью параметра -Force.

Удаляемый профиль также можно указать с помощью объекта профиля:

```powershell
$TmProfile = Get-AzTrafficManagerProfile -Name MyProfile -ResourceGroupName MyRG
Remove-AzTrafficManagerProfile -TrafficManagerProfile $TmProfile [-Force]
```

Эти операции также можно объединить в последовательность:

```powershell
Get-AzTrafficManagerProfile -Name MyProfile -ResourceGroupName MyRG | Remove-AzTrafficManagerProfile [-Force]
```

## <a name="next-steps"></a>Дальнейшие действия

[Мониторинг диспетчера трафика](traffic-manager-monitoring.md)

[Рекомендации по безопасности для диспетчера трафика](traffic-manager-performance-considerations.md)
