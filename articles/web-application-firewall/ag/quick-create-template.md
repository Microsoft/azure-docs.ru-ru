---
title: Краткое руководство. Создание брандмауэра веб-приложения Azure версии 2 в Шлюзе приложений с помощью шаблона Azure Resource Manager
titleSuffix: Azure Application Gateway
description: Узнайте, как с помощью шаблона быстрого запуска Azure Resource Manager (шаблона ARM) создать брандмауэр веб-приложения версии 2 в Шлюзе приложений Azure.
services: web-application-firewall
author: vhorne
ms.service: web-application-firewall
ms.topic: quickstart
ms.date: 09/16/2020
ms.author: victorh
ms.custom: subject-armqs
ms.openlocfilehash: 4c5d5c4cafee9402e2afd7ddc047fd3b68e84540
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92091327"
---
# <a name="quickstart-create-an-azure-waf-v2-on-application-gateway-using-an-arm-template"></a>Краткое руководство. Создание брандмауэра веб-приложения Azure версии 2 в Шлюзе приложений с помощью шаблона ARM

В этом кратком руководстве показано, как с помощью шаблона Azure Resource Manager (шаблона ARM) создать брандмауэр веб-приложения версии 2 в Шлюзе приложений.

[!INCLUDE [About Azure Resource Manager](../../../includes/resource-manager-quickstart-introduction.md)]

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

Если среда соответствует предварительным требованиям и вы знакомы с использованием шаблонов ARM, нажмите кнопку **Развертывание в Azure**. Шаблон откроется на портале Azure.

[![Развертывание в Azure](../../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fag-docs-wafv2%2Fazuredeploy.json)

## <a name="prerequisites"></a>Предварительные требования

- Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) бесплатно.

## <a name="review-the-template"></a>Изучение шаблона

С помощью этого шаблона создается простой брандмауэр веб-приложения версии 2 в Шлюзе приложений Azure. В нем определяются общедоступный IP-адрес, интерфейсный IP-адрес, параметры HTTP, правило с базовым прослушивателем для порта 80 и серверный пул. Для блокировки трафика в серверный пул по типу соответствия IP-адресов создается политика брандмауэра веб-приложения с настраиваемым правилом.

Шаблон, используемый в этом кратком руководстве, взят из [шаблонов быстрого запуска Azure](https://azure.microsoft.com/resources/templates/ag-docs-wafv2/).

:::code language="json" source="~/quickstart-templates/ag-docs-wafv2/azuredeploy.json":::

В шаблоне определено несколько ресурсов Azure:

- [**Microsoft.Network/applicationgateways**](/azure/templates/microsoft.network/applicationgateways);
- [**Microsoft.Network/ApplicationGatewayWebApplicationFirewallPolicies**](/azure/templates/microsoft.network/ApplicationGatewayWebApplicationFirewallPolicies)
- [**Microsoft.Network/publicIPAddresses**](/azure/templates/microsoft.network/publicipaddresses): один для шлюза приложений и два для виртуальных машин;
- [**Microsoft.Network/networkSecurityGroups;**](/azure/templates/microsoft.network/networksecuritygroups)
- [**Microsoft.Network/virtualNetworks**](/azure/templates/microsoft.network/virtualnetworks);
- [**Microsoft.Compute/virtualMachines**](/azure/templates/microsoft.compute/virtualmachines): две виртуальные машины;
- [**Microsoft.Network/networkInterfaces**](/azure/templates/microsoft.network/networkinterfaces): два для виртуальных машин;
- [**Microsoft.Compute/virtualMachine/extensions**](/azure/templates/microsoft.compute/virtualmachines/extensions): для настройки IIS и веб-страниц.

## <a name="deploy-the-template"></a>Развертывание шаблона

Разверните шаблон ARM в Azure:

1. Выберите элемент **Развертывание в Azure**, чтобы войти в Azure и открыть шаблон. С помощью шаблона будут созданы шлюз приложений, сетевая инфраструктура и две виртуальные машины в серверном пуле, где запущены службы IIS.

   [![Развертывание в Azure](../../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fag-docs-wafv2%2Fazuredeploy.json)

2. Выберите или создайте группу ресурсов.
3. Установите флажок **Я принимаю указанные выше условия** и нажмите кнопку **Приобрести**. Для развертывания может потребоваться 10 минут или более.

## <a name="validate-the-deployment"></a>Проверка развертывания

Для создания шлюза приложений не нужно устанавливать службы IIS. Но они устанавливаются на внутренних серверах, чтобы проверить, создан ли брандмауэр веб-приложения версии 2 в шлюзе приложений.

Используйте IIS для тестирования шлюза приложений.

1. На странице **Обзор** найдите общедоступный IP-адрес для шлюза приложений. ![Запишите общедоступный IP-адрес шлюза приложений](../../application-gateway/media/application-gateway-create-gateway-portal/application-gateway-record-ag-address.png). В качестве альтернативы вы можете выбрать **Все ресурсы** и ввести в поисковое поле *myAGPublicIPAddress*, а затем выбрать этот адрес из результатов поиска. Общедоступный IP-адрес отобразится в Azure на странице **Обзор**.
2. Чтобы перейти по общедоступному IP-адресу, скопируйте и вставьте его в адресную строку браузера.
3. Проверьте ответ. Ответ **403 Forbidden** (запрещено) свидетельствует об успешном создании брандмауэра веб-приложения, блокирующего подключения к серверному пулу.
4. Измените настраиваемое правило так, чтобы **разрешить трафик**.
  Выполните следующий скрипт Azure PowerShell, заменив имя группы ресурсов:
   ```azurepowershell
   $rg = "<your resource group name>"
   $AppGW = Get-AzApplicationGateway -Name myAppGateway -ResourceGroupName $rg
   $pol = Get-AzApplicationGatewayFirewallPolicy -Name WafPol01 -ResourceGroupName $rg
   $pol[0].customrules[0].action = "allow"
   $rule = $pol.CustomRules
   Set-AzApplicationGatewayFirewallPolicy -Name WafPol01 -ResourceGroupName $rg -CustomRule $rule
   $AppGW.FirewallPolicy = $pol
   Set-AzApplicationGateway -ApplicationGateway $AppGW
   ```

   Обновите браузер несколько раз, чтобы отобразились сведения о подключениях к myVM1 и myVM2.

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вам уже не нужны ресурсы, созданные с помощью шлюза приложений, удалите группу ресурсов. При этом будут удалены шлюз приложений и все связанные с ним ресурсы.

Чтобы удалить группу ресурсов, вызовите командлет `Remove-AzResourceGroup`:

```azurepowershell-interactive
Remove-AzResourceGroup -Name "<your resource group name>"
```

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Руководство. Создание шлюза приложений с брандмауэром веб-приложения с помощью портала Azure](application-gateway-web-application-firewall-portal.md)