---
title: Настройка настраиваемых правил WAF & наборе правил по умолчанию для передней дверцы Azure
description: Узнайте, как настроить политику WAF, включающую как настраиваемые, так и управляемые правила для существующей конечной точки передней дверцы.
services: web-application-firewall
author: vhorne
ms.service: web-application-firewall
ms.topic: article
ms.date: 09/05/2019
ms.author: victorh
ms.openlocfilehash: 9a0e262db9f5c37189a589eefc451a88dd5ea8c6
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94563415"
---
# <a name="configure-a-web-application-firewall-policy-using-azure-powershell"></a>Настройка политики брандмауэра веб-приложения с помощью Azure PowerShell

Политика брандмауэра веб-приложения Azure (WAF) определяет проверки, необходимые при поступлении запроса на переднюю дверцу.
В этой статье показано, как настроить политику WAF, состоящую из некоторых настраиваемых правил, с включенным набором правил по умолчанию, управляемым Azure.

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

## <a name="prerequisites"></a>Предварительные требования

Прежде чем приступить к настройке политики ограничения скорости, настройте среду PowerShell и создайте профиль передней дверцы.

### <a name="set-up-your-powershell-environment"></a>Настройка среды PowerShell

В Azure PowerShell доступен набор командлетов, которые используют модель [Azure Resource Manager](../../azure-resource-manager/management/overview.md) для управления ресурсами Azure. 

Вы можете установить [Azure PowerShell](/powershell/azure/) на локальном компьютере и использовать его в любом сеансе PowerShell. Выполните инструкции на странице, чтобы войти с помощью учетных данных Azure, и установите команду AZ PowerShell Module.

#### <a name="sign-in-to-azure"></a>Вход в Azure

```
Connect-AzAccount

```
Перед установкой модуля Front Door убедитесь, что у вас установлена текущая версия PowerShellGet. Выполните следующую команду и снова откройте PowerShell.

```
Install-Module PowerShellGet -Force -AllowClobber
``` 

#### <a name="install-azfrontdoor-module"></a>Установка модуля Az.FrontDoor 

```
Install-Module -Name Az.FrontDoor
```
### <a name="create-a-front-door-profile"></a>Создание профиля Front Door

Создайте профиль передней дверцы, следуя инструкциям, приведенным в [кратком руководстве по созданию профиля передней дверцы.](../../frontdoor/quickstart-create-front-door.md)

## <a name="custom-rule-based-on-http-parameters"></a>Настраиваемое правило на основе параметров HTTP

В следующем примере показано, как настроить пользовательское правило с двумя условиями соответствия с помощью [New-азфронтдурвафматчкондитионобжект](/powershell/module/az.frontdoor/new-azfrontdoorwafmatchconditionobject). Запросы относятся к указанному сайту, как определено в источнике ссылок, а строка запроса не содержит "Password". 

```powershell-interactive
$referer = New-AzFrontDoorWafMatchConditionObject -MatchVariable RequestHeader -OperatorProperty Equal -Selector "Referer" -MatchValue "www.mytrustedsites.com/referpage.html"
$password = New-AzFrontDoorWafMatchConditionObject -MatchVariable QueryString -OperatorProperty Contains -MatchValue "password"
$AllowFromTrustedSites = New-AzFrontDoorWafCustomRuleObject -Name "AllowFromTrustedSites" -RuleType MatchRule -MatchCondition $referer,$password -Action Allow -Priority 1
```

## <a name="custom-rule-based-on-http-request-method"></a>Настраиваемое правило на основе метода HTTP-запроса

Создайте метод, блокирующий правило, с помощью [New-азфронтдурвафкустомрулеобжект](/powershell/module/az.frontdoor/new-azfrontdoorwafcustomruleobject) следующим образом:

```powershell-interactive
$put = New-AzFrontDoorWafMatchConditionObject -MatchVariable RequestMethod -OperatorProperty Equal -MatchValue PUT
$BlockPUT = New-AzFrontDoorWafCustomRuleObject -Name "BlockPUT" -RuleType MatchRule -MatchCondition $put -Action Block -Priority 2
```

## <a name="create-a-custom-rule-based-on-size-constraint"></a>Создание настраиваемого правила на основе ограничения размера

В следующем примере создается правило блокировки запросов с URL-адресом, длина которого превышает 100 символов, с помощью Azure PowerShell:
```powershell-interactive
$url = New-AzFrontDoorWafMatchConditionObject -MatchVariable RequestUri -OperatorProperty GreaterThanOrEqual -MatchValue 100
$URLOver100 = New-AzFrontDoorWafCustomRuleObject -Name "URLOver100" -RuleType MatchRule -MatchCondition $url -Action Block -Priority 3
```
## <a name="add-managed-default-rule-set"></a>Добавить набор управляемых правил по умолчанию

В следующем примере создается управляемый набор правил по умолчанию с помощью Azure PowerShell.
```powershell-interactive
$managedRules =  New-AzFrontDoorWafManagedRuleObject -Type DefaultRuleSet -Version 1.0
```
## <a name="configure-a-security-policy"></a>Настройка политики безопасности

С помощью команды `Get-AzResourceGroup` найдите имя группы ресурсов, содержащей профиль Front Door. Затем настройте политику безопасности с созданными правилами в предыдущих шагах, используя [New-азфронтдурвафполици](/powershell/module/az.frontdoor/new-azfrontdoorwafpolicy) в указанной группе ресурсов, которая содержит профиль передней дверцы.

```powershell-interactive
$myWAFPolicy=New-AzFrontDoorWafPolicy -Name $policyName -ResourceGroupName $resourceGroupName -Customrule $AllowFromTrustedSites,$BlockPUT,$URLOver100 -ManagedRule $managedRules -EnabledState Enabled -Mode Prevention
```

## <a name="link-policy-to-a-front-door-front-end-host"></a>Связывание политики с интерфейсным узлом передней дверцы

Свяжите объект политики безопасности с существующим интерфейсным узлом передней дверцы и обновите свойства передней дверцы. Сначала извлеките объект Front дверь, используя [Get-азфронтдур](/powershell/module/Az.FrontDoor/Get-AzFrontDoor).
Затем задайте для свойства внешнего интерфейса *вебаппликатионфиреваллполицилинк* значение *resourceId* из "$myWAFPolicy $", созданного на предыдущем шаге, с помощью [Set-азфронтдур](/powershell/module/Az.FrontDoor/Set-AzFrontDoor). 

В приведенном ниже примере используется имя группы ресурсов *myResourceGroupFD1* с предположением, что вы создали профиль передней дверцы с помощью инструкций, приведенных в статье [Краткое руководство по созданию передней дверцы](../../frontdoor/quickstart-create-front-door.md) . Кроме того, в следующем примере замените $frontDoorName именем профиля передней дверцы. 

```powershell-interactive
   $FrontDoorObjectExample = Get-AzFrontDoor `
     -ResourceGroupName myResourceGroupFD1 `
     -Name $frontDoorName
   $FrontDoorObjectExample[0].FrontendEndpoints[0].WebApplicationFirewallPolicyLink = $myWAFPolicy.Id
   Set-AzFrontDoor -InputObject $FrontDoorObjectExample[0]
 ```

> [!NOTE]
> Свойство *вебаппликатионфиреваллполицилинк* необходимо установить только один раз, чтобы связать политику безопасности с внешним интерфейсом передней дверцы. Последующие обновления политики автоматически применяются к интерфейсной части.

## <a name="next-steps"></a>Дальнейшие действия

- Дополнительные сведения о [передней дверце](../../frontdoor/front-door-overview.md) 
- Дополнительные сведения о [WAF с помощью передней дверцы](afds-overview.md)