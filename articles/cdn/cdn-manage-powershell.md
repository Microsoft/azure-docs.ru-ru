---
title: Управление Azure CDN с помощью PowerShell | Документация Майкрософт
description: Используйте этот учебник, чтобы узнать, как использовать PowerShell для управления аспектами профилей и конечных точек сети доставки содержимого Azure.
services: cdn
documentationcenter: ''
author: asudbring
manager: danielgi
editor: ''
ms.assetid: fb6f57a5-6e26-4847-8fd9-b51fb05a79eb
ms.service: azure-cdn
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 11/20/2019
ms.author: allensu
ms.openlocfilehash: b5fa1cedf4cc5e87d11dc4b5c453d9cb2a307c3c
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "88192664"
---
# <a name="manage-azure-cdn-with-powershell"></a>Управление Azure CDN с помощью PowerShell
PowerShell — это одно из самых гибких средств управления профилями и конечными точками Azure CDN.  PowerShell можно использовать интерактивно или подготовить скрипты для автоматизации задач управления.  В этом руководстве описано несколько распространенных задач по управлению профилями и конечными точками Azure CDN, которые можно выполнять с помощью PowerShell.

## <a name="prerequisites"></a>Предварительные требования

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Чтобы использовать PowerShell для управления профилями и конечными точками Azure CDN, прежде всего нужно установить модуль Azure PowerShell.  Сведения об установке Azure PowerShell и подключении к Azure с помощью командлета `Connect-AzAccount` см. в статье [How to install and configure Azure PowerShell](/powershell/azure/) (Установка и настройка Azure PowerShell).

> [!IMPORTANT]
> Чтобы выполнять командлеты Azure PowerShell, необходимо сначала войти в систему с помощью `Connect-AzAccount`.
> 
> 

## <a name="listing-the-azure-cdn-cmdlets"></a>Получение списка командлетов Azure CDN
Полный список командлетов Azure CDN можно получить с помощью `Get-Command` .

```text
PS C:\> Get-Command -Module Az.Cdn

CommandType     Name                                               Version    Source
-----------     ----                                               -------    ------
Cmdlet          Confirm-AzCdnEndpointProbeURL                      1.4.0      Az.Cdn
Cmdlet          Disable-AzCdnCustomDomain                          1.4.0      Az.Cdn
Cmdlet          Disable-AzCdnCustomDomainHttps                     1.4.0      Az.Cdn
Cmdlet          Enable-AzCdnCustomDomain                           1.4.0      Az.Cdn
Cmdlet          Enable-AzCdnCustomDomainHttps                      1.4.0      Az.Cdn
Cmdlet          Get-AzCdnCustomDomain                              1.4.0      Az.Cdn
Cmdlet          Get-AzCdnEdgeNode                                  1.4.0      Az.Cdn
Cmdlet          Get-AzCdnEndpoint                                  1.4.0      Az.Cdn
Cmdlet          Get-AzCdnEndpointNameAvailability                  1.4.0      Az.Cdn
Cmdlet          Get-AzCdnEndpointResourceUsage                     1.4.0      Az.Cdn
Cmdlet          Get-AzCdnOrigin                                    1.4.0      Az.Cdn
Cmdlet          Get-AzCdnProfile                                   1.4.0      Az.Cdn
Cmdlet          Get-AzCdnProfileResourceUsage                      1.4.0      Az.Cdn
Cmdlet          Get-AzCdnProfileSsoUrl                             1.4.0      Az.Cdn
Cmdlet          Get-AzCdnProfileSupportedOptimizationType          1.4.0      Az.Cdn
Cmdlet          Get-AzCdnSubscriptionResourceUsage                 1.4.0      Az.Cdn
Cmdlet          New-AzCdnCustomDomain                              1.4.0      Az.Cdn
Cmdlet          New-AzCdnDeliveryPolicy                            1.4.0      Az.Cdn
Cmdlet          New-AzCdnDeliveryRule                              1.4.0      Az.Cdn
Cmdlet          New-AzCdnDeliveryRuleAction                        1.4.0      Az.Cdn
Cmdlet          New-AzCdnDeliveryRuleCondition                     1.4.0      Az.Cdn
Cmdlet          New-AzCdnEndpoint                                  1.4.0      Az.Cdn
Cmdlet          New-AzCdnProfile                                   1.4.0      Az.Cdn
Cmdlet          Publish-AzCdnEndpointContent                       1.4.0      Az.Cdn
Cmdlet          Remove-AzCdnCustomDomain                           1.4.0      Az.Cdn
Cmdlet          Remove-AzCdnEndpoint                               1.4.0      Az.Cdn
Cmdlet          Remove-AzCdnProfile                                1.4.0      Az.Cdn
Cmdlet          Set-AzCdnEndpoint                                  1.4.0      Az.Cdn
Cmdlet          Set-AzCdnOrigin                                    1.4.0      Az.Cdn
Cmdlet          Set-AzCdnProfile                                   1.4.0      Az.Cdn
Cmdlet          Start-AzCdnEndpoint                                1.4.0      Az.Cdn
Cmdlet          Stop-AzCdnEndpoint                                 1.4.0      Az.Cdn
Cmdlet          Test-AzCdnCustomDomain                             1.4.0      Az.Cdn
Cmdlet          Unpublish-AzCdnEndpointContent                     1.4.0      Az.Cdn
```

## <a name="getting-help"></a>Получение справки
Справку по любому из этих командлетов можно получить с помощью `Get-Help` .  `Get-Help` описывает назначение и синтаксис командлетов, а для некоторых из них приводит примеры.

```text
PS C:\> Get-Help Get-AzCdnProfile

NAME
    Get-AzCdnProfile

SYNOPSIS
    Gets an Azure CDN profile.


SYNTAX
    Get-AzCdnProfile [-ProfileName <String>] [-ResourceGroupName <String>] [-InformationAction
    <ActionPreference>] [-InformationVariable <String>] [<CommonParameters>]


DESCRIPTION
    Gets an Azure CDN profile and all related information.


RELATED LINKS

REMARKS
    To see the examples, type: "get-help Get-AzCdnProfile -examples".
    For more information, type: "get-help Get-AzCdnProfile -detailed".
    For technical information, type: "get-help Get-AzCdnProfile -full".

```

## <a name="listing-existing-azure-cdn-profiles"></a>Получение списка существующих профилей Azure CDN
Командлет `Get-AzCdnProfile` , запущенный без параметров, возвращает все существующие CDN профили.

```powershell
Get-AzCdnProfile
```

Этот результат можно передавать в командлеты для перечисления.

```powershell
# Output the name of all profiles on this subscription.
Get-AzCdnProfile | ForEach-Object { Write-Host $_.Name }

# Return only **Azure CDN from Verizon** profiles.
Get-AzCdnProfile | Where-Object { $_.Sku.Name -eq "Standard_Verizon" }
```

Также можно получить отдельный профиль, указав имя профиля и группу ресурсов.

```powershell
Get-AzCdnProfile -ProfileName CdnDemo -ResourceGroupName CdnDemoRG
```

> [!TIP]
> Несколько профилей CDN могут иметь одинаковые имена, если они находятся в разных группах ресурсов.  Исключив параметр `ResourceGroupName` , вы получите все профили с указанным именем.
> 
> 

## <a name="listing-existing-cdn-endpoints"></a>Получение списка существующих конечных точек CDN
`Get-AzCdnEndpoint` позволяет получить отдельную конечную точку или все конечные точки в профиле.  

```powershell
# Get a single endpoint.
Get-AzCdnEndpoint -ProfileName CdnDemo -ResourceGroupName CdnDemoRG -EndpointName cdndocdemo

# Get all of the endpoints on a given profile. 
Get-AzCdnEndpoint -ProfileName CdnDemo -ResourceGroupName CdnDemoRG

# Return all of the endpoints on all of the profiles.
Get-AzCdnProfile | Get-AzCdnEndpoint

# Return all of the endpoints in this subscription that are currently running.
Get-AzCdnProfile | Get-AzCdnEndpoint | Where-Object { $_.ResourceState -eq "Running" }
```

## <a name="creating-cdn-profiles-and-endpoints"></a>Создание конечных точек и профилей CDN
Для создания конечных точек и профилей CDN используются командлеты `New-AzCdnProfile` и `New-AzCdnEndpoint`. Поддерживаются такие типы номера SKU:
- Standard_Verizon
- Premium_Verizon
- Custom_Verizon
- Standard_Akamai
- Standard_Microsoft
- Standard_ChinaCdn

```powershell
# Create a new profile
New-AzCdnProfile -ProfileName CdnPoshDemo -ResourceGroupName CdnDemoRG -Sku Standard_Akamai -Location "Central US"

# Create a new endpoint
New-AzCdnEndpoint -ProfileName CdnPoshDemo -ResourceGroupName CdnDemoRG -Location "Central US" -EndpointName cdnposhdoc -OriginName "Contoso" -OriginHostName "www.contoso.com"

# Create a new profile and endpoint (same as above) in one line
New-AzCdnProfile -ProfileName CdnPoshDemo -ResourceGroupName CdnDemoRG -Sku Standard_Akamai -Location "Central US" | New-AzCdnEndpoint -EndpointName cdnposhdoc -OriginName "Contoso" -OriginHostName "www.contoso.com"

```

## <a name="checking-endpoint-name-availability"></a>Проверка доступности имени конечной точки
`Get-AzCdnEndpointNameAvailability` возвращает объект с информацией о том, свободно ли указанное имя конечной точки.

```powershell
# Retrieve availability
$availability = Get-AzCdnEndpointNameAvailability -EndpointName "cdnposhdoc"

# If available, write a message to the console.
If($availability.NameAvailable) { Write-Host "Yes, that endpoint name is available." }
Else { Write-Host "No, that endpoint name is not available." }
```

## <a name="adding-a-custom-domain"></a>Добавление личного домена
`New-AzCdnCustomDomain` добавляет пользовательское имя домена для существующей конечной точки.

> [!IMPORTANT]
> Необходимо настроить запись CNAME у вашего поставщика DNS, как описано в статье [Как сопоставить личный домен с конечной точкой сети доставки содержимого (CDN)](cdn-map-content-to-custom-domain.md).  Это сопоставление можно проверить с помощью `Test-AzCdnCustomDomain`до изменения конечной точки.
> 
> 

```powershell
# Get an existing endpoint
$endpoint = Get-AzCdnEndpoint -ProfileName CdnPoshDemo -ResourceGroupName CdnDemoRG -EndpointName cdnposhdoc

# Check the mapping
$result = Test-AzCdnCustomDomain -CdnEndpoint $endpoint -CustomDomainHostName "cdn.contoso.com"

# Create the custom domain on the endpoint
If($result.CustomDomainValidated){ New-AzCdnCustomDomain -CustomDomainName Contoso -HostName "cdn.contoso.com" -CdnEndpoint $endpoint }
```

## <a name="modifying-an-endpoint"></a>Изменение конечной точки
`Set-AzCdnEndpoint` изменяет существующую конечную точку.

```powershell
# Get an existing endpoint
$endpoint = Get-AzCdnEndpoint -ProfileName CdnPoshDemo -ResourceGroupName CdnDemoRG -EndpointName cdnposhdoc

# Set up content compression
$endpoint.IsCompressionEnabled = $true
$endpoint.ContentTypesToCompress = "text/javascript","text/css","application/json"

# Save the changed endpoint and apply the changes
Set-AzCdnEndpoint -CdnEndpoint $endpoint
```

## <a name="purgingpre-loading-cdn-assets"></a>Очистка и предварительная загрузка ресурсов CDN
`Unpublish-AzCdnEndpointContent` очищает все кэшированные ресурсы, а `Publish-AzCdnEndpointContent` предварительно загружает ресурсы на всех поддерживаемых конечных точках.

```powershell
# Purge some assets.
Unpublish-AzCdnEndpointContent -ProfileName CdnDemo -ResourceGroupName CdnDemoRG -EndpointName cdndocdemo -PurgeContent "/images/kitten.png","/video/rickroll.mp4"

# Pre-load some assets.
Publish-AzCdnEndpointContent -ProfileName CdnDemo -ResourceGroupName CdnDemoRG -EndpointName cdndocdemo -LoadContent "/images/kitten.png","/video/rickroll.mp4"

# Purge everything in /images/ on all endpoints.
Get-AzCdnProfile | Get-AzCdnEndpoint | Unpublish-AzCdnEndpointContent -PurgeContent "/images/*"
```

## <a name="startingstopping-cdn-endpoints"></a>Запуск и остановка конечных точек CDN
`Start-AzCdnEndpoint` и `Stop-AzCdnEndpoint` позволяют запускать и останавливать отдельные конечные точки или группы конечных точек.

```powershell
# Stop the cdndocdemo endpoint
Stop-AzCdnEndpoint -ProfileName CdnDemo -ResourceGroupName CdnDemoRG -EndpointName cdndocdemo

# Stop all endpoints
Get-AzCdnProfile | Get-AzCdnEndpoint | Stop-AzCdnEndpoint

# Start all endpoints
Get-AzCdnProfile | Get-AzCdnEndpoint | Start-AzCdnEndpoint
```

## <a name="creating-standard-rules-engine-policy-and-applying-to-an-existing-cdn-endpoint"></a>Создание политики обработчика стандартных правил и применение ее к существующей конечной точке CDN
`New-AzCdnDeliveryRule`, `New=AzCdnDeliveryRuleCondition` и `New-AzCdnDeliveryRuleAction` могут использоваться для настройки Azure CDN стандартного обработчика правил для Azure CDN из профилей Майкрософт. 

```powershell
# Create a new http to https redirect rule
$Condition=New-AzCdnDeliveryRuleCondition -MatchVariable RequestProtocol -Operator Equal -MatchValue HTTP
$Action=New-AzCdnDeliveryRuleAction -RedirectType Found -DestinationProtocol HTTPS
$HttpToHttpsRedirectRule=New-AzCdnDeliveryRule -Name "HttpToHttpsRedirectRule" -Order 2 -Condition $Condition -Action $Action

# Create a path based Response header modification rule. 
$Cond1=New-AzCdnDeliveryRuleCondition -MatchVariable UrlPath -Operator BeginsWith -MatchValue "/images/"
$Action1=New-AzCdnDeliveryRuleAction -HeaderActionType ModifyResponseHeader -Action Overwrite -HeaderName "Access-Control-Allow-Origin" -Value "*"
$PathBasedCacheOverrideRule=New-AzCdnDeliveryRule -Name "PathBasedCacheOverride" -Order 1 -Condition $Cond1 -Action $action1

# Create a delivery policy with above deliveryRules.
$Policy = New-AzCdnDeliveryPolicy -Description "DeliveryPolicy" -Rule $HttpToHttpsRedirectRule,$UrlRewriteRule

# Update existing endpoint with created delivery policy
$ep = Get-AzCdnEndpoint -EndpointName cdndocdemo -ProfileName CdnDemo -ResourceGroupName CdnDemoRG
$ep.DeliveryPolicy = $Policy
Set-AzCdnEndpoint -CdnEndpoint $ep
```

## <a name="deleting-cdn-resources"></a>Удаление ресурсов CDN
`Remove-AzCdnProfile` и `Remove-AzCdnEndpoint` позволяют удалять профили и конечные точки.

```powershell
# Remove a single endpoint
Remove-AzCdnEndpoint -ProfileName CdnPoshDemo -ResourceGroupName CdnDemoRG -EndpointName cdnposhdoc

# Remove all the endpoints on a profile and skip confirmation (-Force)
Get-AzCdnProfile -ProfileName CdnPoshDemo -ResourceGroupName CdnDemoRG | Get-AzCdnEndpoint | Remove-AzCdnEndpoint -Force

# Remove a single profile
Remove-AzCdnProfile -ProfileName CdnPoshDemo -ResourceGroupName CdnDemoRG
```

## <a name="next-steps"></a>Дальнейшие действия
Узнайте об автоматизации Azure CDN с помощью [.NET](cdn-app-dev-net.md) или [Node.js](cdn-app-dev-node.md).

Сведения о функциях CDN см. в статье [Общие сведения о сети доставки содержимого (CDN) Azure](cdn-overview.md).

