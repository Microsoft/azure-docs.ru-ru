---
title: Повторное создание заголовков HTTP в шлюзе приложений Azure
description: В этой статье содержатся сведения о перезаписи заголовков HTTP в шлюзе приложений Azure с помощью Azure PowerShell
services: application-gateway
author: abshamsft
ms.service: application-gateway
ms.topic: how-to
ms.date: 04/12/2019
ms.author: absha
ms.openlocfilehash: 29ca3aff7d75c7a14bf7b325719924936762d191
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101711694"
---
# <a name="rewrite-http-request-and-response-headers-with-azure-application-gateway---azure-powershell"></a>Перезапись заголовков HTTP-запроса и ответа с помощью шлюза приложений Azure — Azure PowerShell

В этой статье описывается, как использовать Azure PowerShell для настройки экземпляра [SKU шлюза приложений версии 2](./application-gateway-autoscaling-zone-redundant.md) для перезаписи заголовков HTTP в запросах и ответах.

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

## <a name="before-you-begin"></a>Перед началом

- Для выполнения действий, описанных в этой статье, необходимо запустить Azure PowerShell локально. Также необходимо установить параметр AZ Module версии 1.0.0 или более поздней. Запустите `Import-Module Az` , а затем `Get-Module Az` Определите установленную версию. Если вам необходимо выполнить обновление, ознакомьтесь со статьей, посвященной [установке модуля Azure PowerShell](/powershell/azure/install-az-ps). После проверки версии PowerShell выполните командлет `Login-AzAccount`, чтобы создать подключение к Azure.
- Необходимо иметь экземпляр SKU шлюза приложений версии 2. Перезапись заголовков не поддерживается в номере SKU v1. Если у вас нет номера SKU v2, создайте экземпляр [шлюза приложений версии 2 (SKU](./tutorial-autoscale-ps.md) ) перед началом.

## <a name="create-required-objects"></a>Создание необходимых объектов

Чтобы настроить перезапись HTTP-заголовка, необходимо выполнить следующие действия.

1. Создайте объекты, необходимые для перезаписи заголовка HTTP:

   - **Рекуессеадерконфигуратион**: используется для указания полей заголовка запроса, которые требуется перезаписать, и нового значения для заголовков.

   - **Респонсехеадерконфигуратион**: используется для указания полей заголовка ответа, которые требуется перезаписать, и нового значения для заголовков.

   - Набор **действий**: содержит конфигурации заголовков запросов и ответов, указанных ранее.

   - **Условие**: Необязательная конфигурация. Условия перезаписи оценивают содержимое запросов HTTP (S) и ответов. Действие перезаписи будет выполнено, если запрос HTTP (S) или ответ соответствует условию перезаписи.

     При связывании более одного условия с действием действие выполняется только при соблюдении всех условий. Иными словами, операция является логической операцией и.

   - **Ревритеруле**: содержит несколько сочетаний "действие перезаписи" и "Перезапись".

   - **Рулесекуенце**: Необязательная конфигурация, которая помогает определить порядок выполнения правил перезаписи. Эта конфигурация полезна при наличии нескольких правил перезаписи в наборе перезаписи. Сначала выполняется правило перезаписи с более низким значением последовательности правил. Если присвоить одно и то же значение последовательности правил двум правилам перезаписи, порядок выполнения не детерминирован.

     Если вы не укажете Рулесекуенце явным образом, задается значение по умолчанию 100.

   - **Ревритерулесет**: содержит несколько правил перезаписи, которые будут связаны с правилом маршрутизации запросов.

2. Присоедините Ревритерулесет к правилу маршрутизации. Конфигурация перезаписи прикрепляется к исходному прослушивателю с помощью правила маршрутизации. При использовании базового правила маршрутизации конфигурация перезаписи заголовка связана с прослушивателем источника и переписывается глобальным заголовком. При использовании правила маршрутизации на основе пути конфигурация перезаписи заголовка определяется на карте URL-пути. В этом случае он применяется только к определенной области пути сайта.

Можно создать несколько наборов перезаписи заголовков HTTP и применить каждый набор перезаписи к нескольким прослушивателям. Но для конкретного прослушивателя можно применить только один набор перезаписи.

## <a name="sign-in-to-azure"></a>Вход в Azure

```azurepowershell
Connect-AzAccount
Select-AzSubscription -Subscription "<sub name>"
```

## <a name="specify-the-http-header-rewrite-rule-configuration"></a>Укажите конфигурацию правила переопределения для заголовков HTTP

В этом примере мы изменим URL-адрес перенаправления, перезаписав заголовок Location в HTTP-ответе каждый раз, когда заголовок location содержит ссылку на azurewebsites.net. Для этого мы добавим условие для вычисления того, содержит ли заголовок Location в ответе azurewebsites.net. Мы будем использовать шаблон `(https?)://.*azurewebsites.net(.*)$` . И будем использовать в `{http_resp_Location_1}://contoso.com{http_resp_Location_2}` качестве значения заголовка. Это значение заменит *azurewebsites.NET* на *contoso.com* в заголовке Location.

```azurepowershell
$responseHeaderConfiguration = New-AzApplicationGatewayRewriteRuleHeaderConfiguration -HeaderName "Location" -HeaderValue "{http_resp_Location_1}://contoso.com{http_resp_Location_2}"
$actionSet = New-AzApplicationGatewayRewriteRuleActionSet -RequestHeaderConfiguration $requestHeaderConfiguration -ResponseHeaderConfiguration $responseHeaderConfiguration
$condition = New-AzApplicationGatewayRewriteRuleCondition -Variable "http_resp_Location" -Pattern "(https?):\/\/.*azurewebsites\.net(.*)$" -IgnoreCase
$rewriteRule = New-AzApplicationGatewayRewriteRule -Name LocationHeader -ActionSet $actionSet
$rewriteRuleSet = New-AzApplicationGatewayRewriteRuleSet -Name LocationHeaderRewrite -RewriteRule $rewriteRule
```

## <a name="retrieve-the-configuration-of-your-application-gateway"></a>Получение конфигурации шлюза приложений

```azurepowershell
$appgw = Get-AzApplicationGateway -Name "AutoscalingAppGw" -ResourceGroupName "<rg name>"
```

## <a name="retrieve-the-configuration-of-your-request-routing-rule"></a>Получение конфигурации правила маршрутизации запросов

```azurepowershell
$reqRoutingRule = Get-AzApplicationGatewayRequestRoutingRule -Name rule1 -ApplicationGateway $appgw
```

## <a name="update-the-application-gateway-with-the-configuration-for-rewriting-http-headers"></a>Обновление шлюза приложений с помощью конфигурации для перезаписи заголовков HTTP

```azurepowershell
Add-AzApplicationGatewayRewriteRuleSet -ApplicationGateway $appgw -Name LocationHeaderRewrite -RewriteRule $rewriteRuleSet.RewriteRules
Set-AzApplicationGatewayRequestRoutingRule -ApplicationGateway $appgw -Name rule1 -RuleType $reqRoutingRule.RuleType -BackendHttpSettingsId $reqRoutingRule.BackendHttpSettings.Id -HttpListenerId $reqRoutingRule.HttpListener.Id -BackendAddressPoolId $reqRoutingRule.BackendAddressPool.Id -RewriteRuleSetId $rewriteRuleSet.Id
Set-AzApplicationGateway -ApplicationGateway $appgw
```

## <a name="delete-a-rewrite-rule"></a>Удаление правила переопределения

```azurepowershell
$appgw = Get-AzApplicationGateway -Name "AutoscalingAppGw" -ResourceGroupName "<rg name>"
Remove-AzApplicationGatewayRewriteRuleSet -Name "LocationHeaderRewrite" -ApplicationGateway $appgw
$requestroutingrule= Get-AzApplicationGatewayRequestRoutingRule -Name "rule1" -ApplicationGateway $appgw
$requestroutingrule.RewriteRuleSet= $null
set-AzApplicationGateway -ApplicationGateway $appgw
```

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о том, как настроить некоторые распространенные варианты использования, см. в статье [Общие сценарии перезаписи заголовков](./rewrite-http-headers.md).