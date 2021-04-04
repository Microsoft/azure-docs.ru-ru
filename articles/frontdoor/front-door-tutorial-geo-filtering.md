---
title: Руководство. Настройка политики геофильтрации брандмауэра веб-приложения для Azure Front Door
description: Из этого руководства вы узнаете, как создать политику геофильтрации брандмауэра веб-приложения и связать ее с имеющимся узлом внешнего интерфейса Front Door.
services: frontdoor
documentationcenter: ''
author: duongau
ms.service: frontdoor
ms.workload: infrastructure-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 09/14/2020
ms.author: duau
ms.openlocfilehash: f9b5869f7dd472c50ffb2c1c0dce765200f53882
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "91324032"
---
# <a name="tutorial-how-to-set-up-a-geo-filtering-waf-policy-for-your-front-door"></a>Руководство по Руководство по настройке политики геофильтрации брандмауэра веб-приложения (WAF) для службы Front Door
В этом руководстве показано, как с помощью Azure PowerShell создать простую политику геофильтрации и связать ее с имеющимся узлом внешнего интерфейса Front Door. Этот пример политики геофильтрации будет блокировать запросы от других стран или регионов, кроме США.

В этом руководстве описано следующее:
> [!div class="checklist"]
> - определение условий соответствия геофильтрации;
> - добавление условия соответствия геофильтрации для правила;
> - добавление правил в политику;
> - связывание политики WAF с узлом внешнего интерфейса FrontDoor.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Предварительные требования
* Прежде чем начать настройку политики геофильтрации, настройте среду PowerShell и создайте профиль Front Door.
* Создайте профиль Front Door, следуя инструкциям, описанным в статье [Краткое руководство. Создание профиля Front Door для высокодоступного глобального веб-приложения](quickstart-create-front-door.md).

## <a name="define-geo-filtering-match-condition"></a>Определение условий соответствия геофильтрации

Создайте пример условия соответствия, по которому выбираются запросы, поступающие не из региона "США", с использованием [New-AzFrontDoorWafMatchConditionObject](/powershell/module/az.frontdoor/new-azfrontdoorwafmatchconditionobject) в параметрах при создании условия соответствия. [Здесь](front-door-geo-filtering.md) находится двухбуквенный код страны или региона для сопоставления страны или региона.

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

В приведенном ниже примере используется имя группы ресурсов *FrontDoorQS_rg0* при условии, что профиль Front Door создан с помощью инструкций, приведенных в статье [Краткое руководство. Создание профиля Front Door для высокодоступного глобального веб-приложения](quickstart-create-front-door.md). В следующем примере замените имя политики *geoPolicyAllowUSOnly* уникальным именем политики.

```
$geoPolicy = New-AzFrontDoorWafPolicy `
-Name "geoPolicyAllowUSOnly" `
-resourceGroupName FrontDoorQS_rg0 `
-Customrule $nonUSBlockRule  `
-Mode Prevention `
-EnabledState Enabled
```
## <a name="link-waf-policy-to-a-front-door-frontend-host"></a>Связывание политики WAF с узлом внешнего интерфейса Front Door
Свяжите объект политики WAF с имеющимся узлом внешнего интерфейса Front Door и обновите свойства Front Door. 

Для этого сначала извлеките объект Front Door с помощью команды [Get-AzFrontDoor](/powershell/module/az.frontdoor/get-azfrontdoor). 

```
$geoFrontDoorObjectExample = Get-AzFrontDoor -ResourceGroupName FrontDoorQS_rg0
$geoFrontDoorObjectExample[0].FrontendEndpoints[0].WebApplicationFirewallPolicyLink = $geoPolicy.Id
```
Затем установите для свойства внешнего интерфейса WebApplicationFirewallPolicyLink идентификатор ресурса `geoPolicy` с помощью [Set-AzFrontDoor](/powershell/module/az.frontdoor/set-azfrontdoor).

```
Set-AzFrontDoor -InputObject $geoFrontDoorObjectExample[0]
```

> [!NOTE] 
> Достаточно установить свойство WebApplicationFirewallPolicyLink один раз, чтобы связать политику WAF с узлом внешнего интерфейса Front Door. Последующие обновления политики будут автоматически применяться к узлу внешнего интерфейса.

## <a name="clean-up-resources"></a>Очистка ресурсов

На предыдущих шагах вы настроили правило геофильтрации, связанное с политикой WAF. Затем вы связываете политику с узлом внешнего интерфейса Front Door. Если правило геофильтрации или политика WAF больше не нужны, необходимо сначала отменить связь политики и внешнего интерфейса, а потом удалять политику WAF.

:::image type="content" source="media/front-door-geo-filtering/front-door-disassociate-policy.png" alt-text="Отмена связи политики WAF":::

## <a name="next-steps"></a>Дальнейшие действия

Чтобы узнать, как настроить Брандмауэр веб-приложений для Front Door, перейдите к следующему учебнику.

> [!div class="nextstepaction"]
> [Брандмауэр веб-приложения и Front Door](front-door-waf.md)
