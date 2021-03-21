---
title: Настройка политики геофильтрации для брандмауэра веб-приложения, связанного с Azure Front Door Service
description: Из этого учебника вы узнаете, как создать политику геофильтрации и связать ее с имеющимся узлом внешнего интерфейса Front Door.
services: web-application-firewall
author: vhorne
ms.service: web-application-firewall
ms.topic: conceptual
ms.date: 03/10/2020
ms.author: victorh
ms.reviewer: tyao
ms.openlocfilehash: 479b1d8ed1f4238486bb78e33a6139463578dbba
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94563313"
---
# <a name="set-up-a-geo-filtering-waf-policy-for-your-front-door"></a>Настройка политики геофильтрации для брандмауэра веб-приложения (WAF), связанного с Azure Front Door Service

В этом руководстве показано, как с помощью Azure PowerShell создать простую политику геофильтрации и связать ее с имеющимся узлом внешнего интерфейса Front Door. Этот пример политики геофильтрации будет блокировать запросы от других стран или регионов, кроме США.

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

## <a name="prerequisites"></a>Предварительные требования

Прежде чем начать настройку политики геофильтрации, настройте среду PowerShell и создайте профиль Front Door.
### <a name="set-up-your-powershell-environment"></a>Настройка среды PowerShell
В Azure PowerShell доступен набор командлетов, которые используют модель [Azure Resource Manager](../../azure-resource-manager/management/overview.md) для управления ресурсами Azure. 

Вы можете установить [Azure PowerShell](/powershell/azure/) на локальном компьютере и использовать его в любом сеансе PowerShell. Следуя инструкциям на странице, войдите с учетными данными Azure и установите модуль Az PowerShell.

#### <a name="connect-to-azure-with-an-interactive-dialog-for-sign-in"></a>Подключение к Azure с помощью интерактивного диалогового окна для входа

```
Install-Module -Name Az
Connect-AzAccount
```
Убедитесь, что у вас установлена текущая версия PowerShellGet. Выполните следующую команду и снова откройте PowerShell.

```
Install-Module PowerShellGet -Force -AllowClobber
``` 
#### <a name="install-azfrontdoor-module"></a>Установка модуля Az.FrontDoor 

```
Install-Module -Name Az.FrontDoor
```

### <a name="create-a-front-door-profile"></a>Создание профиля Front Door

Создайте профиль Front Door, следуя инструкциям, описанным в статье [Краткое руководство. Создание профиля Front Door для высокодоступного глобального веб-приложения](../../frontdoor/quickstart-create-front-door.md).

## <a name="define-geo-filtering-match-condition"></a>Определение условий соответствия геофильтрации

Создайте пример условия соответствия, по которому выбираются запросы, поступающие не из региона "США", с использованием [New-AzFrontDoorWafMatchConditionObject](/powershell/module/az.frontdoor/new-azfrontdoorwafmatchconditionobject) в параметрах при создании условия соответствия. В статье [Сведения о геофильтрации в домене для Azure Front Door](waf-front-door-geo-filtering.md) приводятся двухбуквенные коды для разных стран и регионов.

```azurepowershell-interactive
$nonUSGeoMatchCondition = New-AzFrontDoorWafMatchConditionObject `
-MatchVariable RemoteAddr `
-OperatorProperty GeoMatch `
-NegateCondition $true `
-MatchValue "US"
```
 
## <a name="add-geo-filtering-match-condition-to-a-rule-with-action-and-priority"></a>Добавление условия соответствия геофильтрации к правилу с действием и приоритетом

Создайте объект CustomRule `nonUSBlockRule` на основе условия соответствия, действия и приоритета с помощью [New-AzFrontDoorWafCustomRuleObject](/powershell/module/az.frontdoor/new-azfrontdoorwafcustomruleobject).  CustomRule может содержать несколько MatchCondition.  В этом примере для действия устанавливается блокировка, а для приоритета — значение 1 (самый высокий приоритет).

```
$nonUSBlockRule = New-AzFrontDoorWafCustomRuleObject `
-Name "geoFilterRule" `
-RuleType MatchRule `
-MatchCondition $nonUSGeoMatchCondition `
-Action Block `
-Priority 1
```

## <a name="add-rules-to-a-policy"></a>Добавление правил в политику

С помощью команды `Get-AzResourceGroup` найдите имя группы ресурсов, содержащей профиль Front Door. Затем создайте объект политики `geoPolicy`, содержащий `nonUSBlockRule`, используя команду [New-AzFrontDoorWafPolicy](/powershell/module/az.frontdoor/new-azfrontdoorwafpolicy) в указанной группе ресурсов, содержащей профиль Front Door. Укажите уникальное имя для политики геофильтрации. 

В следующем примере используется имя группы ресурсов *myResourceGroupFD1*, а также предполагается, что профиль Front Door создан с помощью инструкций, приведенных в статье [Краткое руководство. Создание профиля Front Door для высокодоступного глобального веб-приложения](../../frontdoor/quickstart-create-front-door.md). В следующем примере замените имя политики *geoPolicyAllowUSOnly* уникальным именем политики.

```
$geoPolicy = New-AzFrontDoorWafPolicy `
-Name "geoPolicyAllowUSOnly" `
-resourceGroupName myResourceGroupFD1 `
-Customrule $nonUSBlockRule  `
-Mode Prevention `
-EnabledState Enabled
```

## <a name="link-waf-policy-to-a-front-door-frontend-host"></a>Связывание политики WAF с узлом внешнего интерфейса Front Door

Свяжите объект политики WAF с имеющимся узлом внешнего интерфейса Front Door и обновите свойства Front Door. 

Для этого сначала извлеките объект Front Door с помощью команды [Get-AzFrontDoor](/powershell/module/az.frontdoor/get-azfrontdoor). 

```
$geoFrontDoorObjectExample = Get-AzFrontDoor -ResourceGroupName myResourceGroupFD1
$geoFrontDoorObjectExample[0].FrontendEndpoints[0].WebApplicationFirewallPolicyLink = $geoPolicy.Id
```

Затем установите для свойства внешнего интерфейса WebApplicationFirewallPolicyLink идентификатор ресурса `geoPolicy` с помощью [Set-AzFrontDoor](/powershell/module/az.frontdoor/set-azfrontdoor).

```
Set-AzFrontDoor -InputObject $geoFrontDoorObjectExample[0]
```

> [!NOTE] 
> Достаточно установить свойство WebApplicationFirewallPolicyLink один раз, чтобы связать политику WAF с узлом внешнего интерфейса Front Door. Последующие обновления политики будут автоматически применяться к узлу внешнего интерфейса.

## <a name="next-steps"></a>Дальнейшие действия

- Сведения о [Брандмауэр веб-приложения Azure](../overview.md).
- Дополнительные сведения о [создании Front Door](../../frontdoor/quickstart-create-front-door.md).