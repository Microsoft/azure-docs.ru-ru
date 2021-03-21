---
title: Настройка пользовательских ответов для брандмауэра веб-приложения (WAF) с помощью передней дверцы Azure
description: Узнайте, как настроить пользовательский код ответа и сообщение, когда WAF блокирует запрос.
services: web-application-firewall
author: vhorne
ms.service: web-application-firewall
ms.topic: article
ms.date: 06/10/2020
ms.author: victorh
ms.reviewer: tyao
ms.openlocfilehash: 8fc6e71494df36cd6f823661b18e4a3d8ce2938c
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94563687"
---
# <a name="configure-a-custom-response-for-azure-web-application-firewall-waf"></a>Настройка пользовательского ответа для брандмауэра веб-приложения Azure (WAF)

По умолчанию, когда WAF блокирует запрос из-за сопоставленного правила, он возвращает код состояния 403 с **запросом блокирует** сообщение. Сообщение по умолчанию также включает строку отслеживания ссылок, которую можно использовать для связи с [записями журнала](./waf-front-door-monitor.md) для запроса.  Вы можете настроить код пользовательского состояния отклика и пользовательское сообщение со строкой ссылки для вашего варианта использования. В этой статье описывается, как настроить настраиваемую страницу ответа, если запрос заблокирован WAF.

## <a name="configure-custom-response-status-code-and-message-use-portal"></a>Настройка кода состояния настраиваемого ответа и используемого сообщения на портале

Вы можете настроить код состояния пользовательского отклика и его текст в разделе "параметры политики" на портале WAF.

:::image type="content" source="../media/waf-front-door-configure-custom-response-code/custom-response-settings.png" alt-text="Параметры политики WAF":::

В приведенном выше примере мы сохранили код ответа как 403 и настроили короткое сообщение "свяжитесь с нами", как показано на рисунке ниже:

:::image type="content" source="../media/waf-front-door-configure-custom-response-code/custom-response.png" alt-text="Пример настраиваемого ответа":::

"{{Azure-ref}}" вставляет уникальную строку ссылки в текст ответа. Значение соответствует полю Траккингреференце в `FrontdoorAccessLog` `FrontdoorWebApplicationFirewallLog` журналах и.

## <a name="configure-custom-response-status-code-and-message-use-powershell"></a>Настройка кода состояния пользовательского отклика и сообщения использование PowerShell

### <a name="set-up-your-powershell-environment"></a>Настройка среды PowerShell

В Azure PowerShell доступен набор командлетов, которые используют модель [Azure Resource Manager](../../azure-resource-manager/management/overview.md) для управления ресурсами Azure. 

Вы можете установить [Azure PowerShell](/powershell/azure/) на локальном компьютере и использовать его в любом сеансе PowerShell. Следуя инструкциям на странице, войдите с учетными данными Azure и установите модуль Az PowerShell.

### <a name="connect-to-azure-with-an-interactive-dialog-for-sign-in"></a>Подключение к Azure с помощью интерактивного диалогового окна для входа

```
Connect-AzAccount
Install-Module -Name Az

```
Убедитесь, что у вас установлена текущая версия PowerShellGet. Выполните следующую команду и снова откройте PowerShell.
```
Install-Module PowerShellGet -Force -AllowClobber
``` 
### <a name="install-azfrontdoor-module"></a>Установка модуля Az.FrontDoor 

```
Install-Module -Name Az.FrontDoor
```

### <a name="create-a-resource-group"></a>Создание группы ресурсов

В Azure выделите связанные ресурсы группе ресурсов. Здесь мы создадим группу ресурсов с помощью [New-азресаурцеграуп](/powershell/module/Az.resources/new-Azresourcegroup).

```azurepowershell-interactive
New-AzResourceGroup -Name myResourceGroupWAF
```

### <a name="create-a-new-waf-policy-with-custom-response"></a>Создание новой политики WAF с настраиваемым ответом 

Ниже приведен пример создания новой политики WAF с кодом состояния настраиваемого ответа, равным 405, и сообщение **заблокировано.** с помощью [New-азфронтдурвафполици](/powershell/module/az.frontdoor/new-azfrontdoorwafpolicy)

```azurepowershell
# WAF policy setting
New-AzFrontDoorWafPolicy `
-Name myWAFPolicy `
-ResourceGroupName myResourceGroupWAF `
-EnabledState enabled `
-Mode Detection `
-CustomBlockResponseStatusCode 405 `
-CustomBlockResponseBody "<html><head><title>You are blocked.</title></head><body></body></html>"
```

Измените пользовательский код ответа или параметры текста ответа существующей политики WAF с помощью [Update-азфронтдурфиреваллполици](/powershell/module/az.frontdoor/Update-AzFrontDoorWafPolicy).

```azurepowershell
# modify WAF response code
Update-AzFrontDoorFireWallPolicy `
-Name myWAFPolicy `
-ResourceGroupName myResourceGroupWAF `
-EnabledState enabled `
-Mode Detection `
-CustomBlockResponseStatusCode 403
```

```azurepowershell
# modify WAF response body
Update-AzFrontDoorFireWallPolicy `
-Name myWAFPolicy `
-ResourceGroupName myResourceGroupWAF `
-CustomBlockResponseBody "<html><head><title>Forbidden</title></head><body>{{azure-ref}}</body></html>"
```

## <a name="next-steps"></a>Дальнейшие действия
- Дополнительные сведения о [брандмауэре веб-приложения с помощью передней дверцы Azure](../afds/afds-overview.md)