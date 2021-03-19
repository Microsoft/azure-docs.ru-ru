---
title: Связывание политики брандмауэра веб-приложения с существующим шлюзом приложений Azure
description: Узнайте, как связать политику брандмауэра веб-приложения с существующим шлюзом приложений Azure.
services: web-application-firewall
ms.topic: article
author: vhorne
ms.service: web-application-firewall
ms.date: 10/25/2019
ms.author: victorh
ms.openlocfilehash: 1ed2e0cf8cc8cd841d8779462d62ba4852774a3a
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "74083904"
---
# <a name="associate-a-waf-policy-with-an-existing-application-gateway"></a>Связывание политики WAF с существующим шлюзом приложений

Вы можете использовать Azure PowerShell для [создания политики WAF](create-waf-policy-ag.md), но у вас уже может быть шлюз приложений и необходимо связать с ним политику WAF. В этой статье вы просто выполните эти действия. Вы создаете политику WAF и связываете ее с уже существующим шлюзом приложений. 

1. Получите шлюз приложений и политику брандмауэра. Если у вас нет существующей политики брандмауэра, см. шаг 2. 

   ```azurepowershell-interactive
      Connect-AzAccount
      Select-AzSubscription -Subscription "<sub id>"`

      #Get Application Gateway and existing policy object or create new`
      $gw = Get-AzApplicationGateway -Name <Waf v2> -ResourceGroupName <RG name>`
      $policy = Get-AzApplicationGatewayFirewallPolicy -Name <policy name> -ResourceGroupName <RG name>`
   ```

2. Используемых Создайте политику брандмауэра.

   ```azurepowershell-interactive
      New-AzApplicationGatewayFirewallPolicy -Name <policy name> -ResourceGroupName <RG name>'
      $policy = Get-AzApplicationGatewayFirewallPolicy -Name <policy name> -ResourceGroupName <RG name>`
   ```
   > [!NOTE]
   > Если вы создаете эту политику WAF для перехода из конфигурации WAF в политику WAF, то эта политика должна быть точной копией старой конфигурации. Это означает, что каждое исключение, настраиваемое правило, отключенная группа правил и т. д. должны быть точно такими же, как и в конфигурации WAF.
3. Используемых Вы можете настроить политику WAF в соответствии со своими потребностями. Сюда входят пользовательские правила, отключение правил или групп правил, исключения, установка ограничений на передачу файлов и т. д. Если пропустить этот шаг, будут выбраны все значения по умолчанию. 
   
4. Сохраните политику и подключите ее к шлюзу приложений. 
   
   ```azurepowershell-interactive
      #Save the policy itself
      Set-AzApplicationGatewayFirewallPolicy -InputObject $policy`
   
      #Attach the policy to an Application Gateway
      $gw.FirewallPolicy = $policy`
   
      #Save the Application Gateway
      Set-AzApplicationGateway -ApplicationGateway $gw`
   ```

## <a name="next-steps"></a>Дальнейшие действия
[Сведения о настраиваемых правилах.](configure-waf-custom-rules.md)
