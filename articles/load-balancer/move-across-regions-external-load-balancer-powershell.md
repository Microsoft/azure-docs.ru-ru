---
title: Перемещение внешних Load Balancer Azure в другой регион Azure с помощью Azure PowerShell
description: Используйте шаблон Azure Resource Manager, чтобы переместить внешние Load Balancer Azure из одного региона Azure в другой с помощью Azure PowerShell.
author: asudbring
ms.service: load-balancer
ms.topic: how-to
ms.date: 09/17/2019
ms.author: allensu
ms.openlocfilehash: 94cb46536bcf029a9e71a7238772ccc7b186b1dc
ms.sourcegitcommit: 15d27661c1c03bf84d3974a675c7bd11a0e086e6
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/09/2021
ms.locfileid: "102500280"
---
# <a name="move-azure-external-load-balancer-to-another-region-using-azure-powershell"></a>Перемещение внешних Load Balancer Azure в другой регион с помощью Azure PowerShell

Существуют различные сценарии, в которых необходимо переместить имеющуюся внешнюю подсистему балансировки нагрузки из одного региона в другой. Например, может потребоваться создать внешнюю подсистему балансировки нагрузки с той же конфигурацией для тестирования. При планировании аварийного восстановления может также потребоваться переместить внешнюю подсистему балансировки нагрузки в другой регион.

Внешние подсистемы балансировки нагрузки Azure невозможно переместить из одного региона в другой. Однако можно использовать шаблон Azure Resource Manager для экспорта существующей конфигурации и общедоступного IP-адреса внешней подсистемы балансировки нагрузки.  Затем можно разместить ресурс в другом регионе, экспортировав подсистему балансировки нагрузки и общедоступный IP-адрес в шаблон, изменив параметры в соответствии с регионом назначения, а затем развернув шаблоны в новом регионе.  Дополнительные сведения о Resource Manager и шаблонах см. в разделе [Экспорт групп ресурсов в шаблоны](../azure-resource-manager/management/manage-resource-groups-powershell.md#export-resource-groups-to-templates)


## <a name="prerequisites"></a>Предварительные требования

- Убедитесь, что внешняя подсистема балансировки нагрузки Azure находится в регионе Azure, из которого вы хотите переместиться.

- Внешние подсистемы балансировки нагрузки Azure нельзя перемещать между регионами.  Необходимо связать новую подсистему балансировки нагрузки с ресурсами в целевом регионе.

- Чтобы экспортировать конфигурацию внешней подсистемы балансировки нагрузки и развернуть шаблон для создания внешней подсистемы балансировки нагрузки в другом регионе, вам потребуется роль "участник сети" или выше.
   
- Определите структуру сети в исходном регионе и все ресурсы, которые вы сейчас используете. Этот макет включает, но не ограничен подсистемами балансировки нагрузки, группами безопасности сети, общедоступными IP-адресами и виртуальными сетями.

- Убедитесь, что подписка Azure позволяет создавать внешние подсистемы балансировки нагрузки в используемом целевом регионе. Свяжитесь со службой поддержки, чтобы включить необходимые квоты.

- Убедитесь, что у вашей подписки достаточно ресурсов для поддержки добавления подсистем балансировки нагрузки для этого процесса.  См. раздел [Подписка Azure, пределы, квоты и ограничения службы](../azure-resource-manager/management/azure-subscription-service-limits.md#networking-limits) .


## <a name="prepare-and-move"></a>Подготовка и перемещение
Ниже показано, как подготовить внешнюю подсистему балансировки нагрузки для перемещения с помощью шаблона диспетчер ресурсов и переместить конфигурацию внешней подсистемы балансировки нагрузки в целевую область с помощью Azure PowerShell.  В рамках этого процесса должна быть включена общедоступная IP-конфигурация внешней подсистемы балансировки нагрузки, которая должна быть выполнена перед перемещением внешней подсистемы балансировки нагрузки.


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

### <a name="export-the-public-ip-template-and-deploy-from-azure-powershell"></a>Экспорт шаблона общедоступного IP-адреса и развертывание из Azure PowerShell

1. С помощью команды [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount) войдите в подписку Azure и следуйте инструкциям на экране:
    
    ```azurepowershell-interactive
    Connect-AzAccount
    ```
2. Получите идентификатор ресурса общедоступного IP-адреса, который необходимо переместить в целевой регион, и включите его в переменную с помощью команды [Get-AzPublicIPAddress](/powershell/module/az.network/get-azpublicipaddress):

    ```azurepowershell-interactive
    $sourcePubIPID = (Get-AzPublicIPaddress -Name <source-public-ip-name> -ResourceGroupName <source-resource-group-name>).Id

    ```
3. Экспортируйте общедоступный исходный IP-адрес в JSON-файл в каталог, в котором выполняется команда [Export — азресаурцеграуп](/powershell/module/az.resources/export-azresourcegroup):
   
   ```azurepowershell-interactive
   Export-AzResourceGroup -ResourceGroupName <source-resource-group-name> -Resource $sourceVNETID -IncludeParameterDefaultValue
   ```

4. Скачанный файл будет называться именем группы ресурсов, из которой был экспортирован ресурс.  Выберите файл, экспортированный из команды, с именем **\<resource-group-name>.json**, и откройте его в любом редакторе:
   
   ```azurepowershell
   notepad.exe <source-resource-group-name>.json
   ```

5. Чтобы изменить параметр имени общедоступного IP-адреса, измените значение свойства **defaultValue** имени исходного общедоступного IP-адреса на имя целевого общедоступного IP-адреса и убедитесь, что имя указано в кавычках:
    
    ```json
        {
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "publicIPAddresses_myVM1pubIP_name": {
        "defaultValue": "<target-publicip-name>",
        "type": "String"
        }
    }

    ```

6. Чтобы изменить целевой регион, в который будет перемещен общедоступный IP-адрес, измените свойство **location** в разделе ресурсов:

    ```json
            "resources": [
            {
            "type": "Microsoft.Network/publicIPAddresses",
            "apiVersion": "2019-06-01",
            "name": "[parameters('publicIPAddresses_myPubIP_name')]",
            "location": "<target-region>",
            "sku": {
                "name": "Basic",
                "tier": "Regional"
            },
            "properties": {
                "provisioningState": "Succeeded",
                "resourceGuid": "7549a8f1-80c2-481a-a073-018f5b0b69be",
                "ipAddress": "52.177.6.204",
                "publicIPAddressVersion": "IPv4",
                "publicIPAllocationMethod": "Dynamic",
                "idleTimeoutInMinutes": 4,
                "ipTags": []
               }
               }
             ]             
    ```
  
7. Чтобы получить коды расположения регионов, можно использовать командлет Azure PowerShell [Get-AzLocation](/powershell/module/az.resources/get-azlocation), выполнив следующую команду:

    ```azurepowershell-interactive

    Get-AzLocation | format-table
    
    ```
8. Кроме того, при необходимости можно изменить другие параметры в шаблоне:

    * **SKU** — вы можете изменить номер SKU общедоступного IP-адреса в конфигурации с уровня "Стандартный" на "Базовый" и наоборот, изменив значение свойства **sku** > **name** в файле **\<resource-group-name>.json**:

         ```json
            "resources": [
                   {
                    "type": "Microsoft.Network/publicIPAddresses",
                    "apiVersion": "2019-06-01",
                    "name": "[parameters('publicIPAddresses_myPubIP_name')]",
                    "location": "<target-region>",
                    "sku": {
                        "name": "Basic",
                        "tier": "Regional"
                    },
         ```

         Дополнительные сведения о различиях между общедоступными IP-адресами уровня "Базовый" и "Стандартный" см. в разделе [Создание, изменение или удаление общедоступного IP-адреса](../virtual-network/virtual-network-public-ip-address.md).

    * **Метод выделения общедоступного IP-адреса** и **Время ожидания простоя**. Вы можете изменить оба этих параметра в шаблоне, изменив свойство **PublicIPAllocationMethod** со значения **Dynamic** на значение **Static** или со значения **Static** на значение **Dynamic**. Время ожидания при простое можно изменить, изменив свойство **IdleTimeoutInMinutes** на требуемый объем.  Значение по умолчанию — **4**:

         ```json
         "resources": [
                {
                "type": "Microsoft.Network/publicIPAddresses",
                "apiVersion": "2019-06-01",
                "name": "[parameters('publicIPAddresses_myPubIP_name')]",
                "location": "<target-region>",
                  "sku": {
                  "name": "Basic",
                  "tier": "Regional"
                 },
                "properties": {
                "provisioningState": "Succeeded",
                "resourceGuid": "7549a8f1-80c2-481a-a073-018f5b0b69be",
                "ipAddress": "52.177.6.204",
                "publicIPAddressVersion": "IPv4",
                "publicIPAllocationMethod": "Dynamic",
                "idleTimeoutInMinutes": 4,
                "ipTags": []
                   }
                }            
         ```

        Дополнительные сведения о методах выделения и значениях времени ожидания при простое см. в разделе [Создание, изменение или удаление общедоступного IP-адреса](../virtual-network/virtual-network-public-ip-address.md).


9. Сохраните файл **\<resource-group-name>.json**.

10. Создайте группу ресурсов в целевом регионе для развертывания целевого общедоступного IP-адреса с помощью команды [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup).
    
    ```azurepowershell-interactive
    New-AzResourceGroup -Name <target-resource-group-name> -location <target-region>
    ```
11. Разверните измененный файл **\<resource-group-name>.json** в группе ресурсов, созданной на предыдущем шаге, с помощью команды [New-AzResourceGroupDeployment](/powershell/module/az.resources/new-azresourcegroupdeployment):

    ```azurepowershell-interactive

    New-AzResourceGroupDeployment -ResourceGroupName <target-resource-group-name> -TemplateFile <source-resource-group-name>.json
    
    ```

12. Чтобы проверить, созданы ли ресурсы в целевом регионе, используйте команду [Get-AzResourceGroup](/powershell/module/az.resources/get-azresourcegroup) и [Get-AzPublicIPAddress](/powershell/module/az.network/get-azpublicipaddress):
    
    ```azurepowershell-interactive

    Get-AzResourceGroup -Name <target-resource-group-name>

    ```

    ```azurepowershell-interactive

    Get-AzPublicIPAddress -Name <target-publicip-name> -ResourceGroupName <target-resource-group-name>

    ```

### <a name="export-the-external-load-balancer-template-and-deploy-from-azure-powershell"></a>Экспорт шаблона внешней подсистемы балансировки нагрузки и развертывание из Azure PowerShell

1. С помощью команды [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount) войдите в подписку Azure и следуйте инструкциям на экране:
    
    ```azurepowershell-interactive
    Connect-AzAccount
    ```

2. Получите идентификатор ресурса внешней подсистемы балансировки нагрузки, которую необходимо переместить в целевой регион, и поместите ее в переменную с помощью команды [Get-азлоадбаланцер](/powershell/module/az.network/get-azloadbalancer):

    ```azurepowershell-interactive
    $sourceExtLBID = (Get-AzLoadBalancer -Name <source-external-lb-name> -ResourceGroupName <source-resource-group-name>).Id

    ```
3. Экспортируйте конфигурацию внешнего подсистемы балансировки нагрузки исходного кода в JSON-файл в каталог, в котором выполняется команда [Export — азресаурцеграуп](/powershell/module/az.resources/export-azresourcegroup):
   
   ```azurepowershell-interactive
   Export-AzResourceGroup -ResourceGroupName <source-resource-group-name> -Resource $sourceExtLBID -IncludeParameterDefaultValue
   ```
4. Скачанный файл будет называться именем группы ресурсов, из которой был экспортирован ресурс.  Выберите файл, экспортированный из команды, с именем **\<resource-group-name>.json**, и откройте его в любом редакторе:
   
   ```azurepowershell
   notepad.exe <source-resource-group-name>.json
   ```

5. Чтобы изменить параметр имени внешней подсистемы балансировки нагрузки, измените значение свойства **DefaultValue** в поле имя внешней подсистемы балансировки нагрузки на имя целевой внешней подсистемы балансировки нагрузки и убедитесь, что имя находится в кавычках:

    ```json
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
        "loadBalancers_myLoadbalancer_ext_name": {
        "defaultValue": "<target-external-lb-name>",
        "type": "String"
            },
        "publicIPAddresses_myPubIP_in_externalid": {
        "defaultValue": "<target-publicIP-resource-ID>",
        "type": "String"
            },

    ```

6.  Чтобы изменить значение целевого общедоступного IP-адреса, который был перемещен выше, сначала необходимо получить идентификатор ресурса, а затем скопировать и вставить его в **\<resource-group-name> JSON** -файл.  Чтобы получить идентификатор, используйте команду [Get-азпублиЦипаддресс](/powershell/module/az.network/get-azpublicipaddress):

    ```azurepowershell-interactive
    $targetPubIPID = (Get-AzPublicIPaddress -Name <target-public-ip-name> -ResourceGroupName <target-resource-group-name>).Id
    ```
    Введите переменную и нажмите клавишу ВВОД, чтобы отобразить идентификатор ресурса.  Выделите путь к ИДЕНТИФИКАТОРу и скопируйте его в буфер обмена:

    ```powershell
    PS C:\> $targetPubIPID
    /subscriptions/7668d659-17fc-4ffd-85ba-9de61fe977e8/resourceGroups/myResourceGroupLB-Move/providers/Microsoft.Network/publicIPAddresses/myPubIP-in-move
    ```

7.  В **\<resource-group-name> JSON** -файле вставьте **идентификатор ресурса** из переменной вместо **DEFAULTVALUE** во второй параметр для внешнего идентификатора public IP, чтобы заключить путь в кавычки:

    ```json
            "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
            "contentVersion": "1.0.0.0",
            "parameters": {
            "loadBalancers_myLoadbalancer_ext_name": {
            "defaultValue": "<target-external-lb-name>",
            "type": "String"
            },
            "publicIPAddresses_myPubIP_in_externalid": {
            "defaultValue": "<target-publicIP-resource-ID>",
            "type": "String"
            },

    ```

8.  Если вы настроили исходящие правила NAT и исходящего трафика для балансировщика нагрузки, в этом файле будет указана третья запись для внешнего идентификатора исходящего общедоступного IP-адреса.  Повторите описанные выше действия в **целевом регионе** , чтобы получить идентификатор исходящего общедоступного iP-адреса и вставить эту запись в **\<resource-group-name> JSON** -файл:

    ```json
            "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
            "contentVersion": "1.0.0.0",
            "parameters": {
                "loadBalancers_myLoadbalancer_ext_name": {
                "defaultValue": "<target-external-lb-name>",
                "type": "String"
            },
                "publicIPAddresses_myPubIP_in_externalid": {
                "defaultValue": "<target-publicIP-resource-ID>",
                "type": "String"
            },
                "publicIPAddresses_myPubIP_out_externalid": {
                "defaultValue": "<target-publicIP-outbound-resource-ID>",
                "type": "String"
            }
        },
    ```

10. Чтобы изменить целевой регион, в который будет перемещена конфигурация внешней подсистемы балансировки нагрузки, измените свойство **Location** в разделе **ресурсы** в файле **\<resource-group-name> . JSON** :

    ```json
        "resources": [
            {
                "type": "Microsoft.Network/loadBalancers",
                "apiVersion": "2019-06-01",
                "name": "[parameters('loadBalancers_myLoadBalancer_name')]",
                "location": "<target-external-lb-region>",
                "sku": {
                    "name": "Standard",
                    "tier": "Regional"
                },
    ```

11. Чтобы получить коды расположения регионов, можно использовать командлет Azure PowerShell [Get-AzLocation](/powershell/module/az.resources/get-azlocation), выполнив следующую команду:

    ```azurepowershell-interactive

    Get-AzLocation | format-table
    
    ```
12. Кроме того, при необходимости можно изменить другие параметры в шаблоне:
    
    * **SKU** . номер SKU внешней подсистемы балансировки нагрузки можно изменить в конфигурации с Standard на Basic или Basic на Standard, изменив   >  свойство **имя** SKU в **\<resource-group-name> JSON** -файле:

        ```json
        "resources": [
        {
            "type": "Microsoft.Network/loadBalancers",
            "apiVersion": "2019-06-01",
            "name": "[parameters('loadBalancers_myLoadBalancer_name')]",
            "location": "<target-external-lb-region>",
            "sku": {
                "name": "Standard",
                "tier": "Regional"
            },
        ```
      Дополнительные сведения о различиях между подсистемами балансировки нагрузки уровня "базовый" и "Стандартный" см. в статье [обзор Load Balancer (цен. Категория "Стандартный") Azure](./load-balancer-overview.md) .

    * **Правила балансировки нагрузки** . в конфигурацию можно добавить или удалить правила балансировки нагрузки, добавив или удалив записи в разделе **лоадбаланЦингрулес** **\<resource-group-name> JSON** -файла:

        ```json
        "loadBalancingRules": [
                    {
                        "name": "myInboundRule",
                        "etag": "W/\"39e5e9cd-2d6d-491f-83cf-b37a259d86b6\"",
                        "properties": {
                            "provisioningState": "Succeeded",
                            "frontendIPConfiguration": {
                                "id": "[concat(resourceId('Microsoft.Network/loadBalancers', parameters('loadBalancers_myLoadBalancer_name')), '/frontendIPConfigurations/myfrontendIPinbound')]"
                            },
                            "frontendPort": 80,
                            "backendPort": 80,
                            "enableFloatingIP": false,
                            "idleTimeoutInMinutes": 4,
                            "protocol": "Tcp",
                            "enableTcpReset": false,
                            "loadDistribution": "Default",
                            "disableOutboundSnat": true,
                            "backendAddressPool": {
                                "id": "[concat(resourceId('Microsoft.Network/loadBalancers', parameters('loadBalancers_myLoadBalancer_name')), '/backendAddressPools/myBEPoolInbound')]"
                            },
                            "probe": {
                                "id": "[concat(resourceId('Microsoft.Network/loadBalancers', parameters('loadBalancers_myLoadBalancer_name')), '/probes/myHTTPProbe')]"
                            }
                        }
                    }
                ]
        ```
       Дополнительные сведения о правилах балансировки нагрузки см [. в разделе что такое Azure Load Balancer?](./load-balancer-overview.md)

    * **Пробы** . Вы можете добавить или удалить пробу для балансировщика нагрузки в конфигурации, добавив или удалив записи в разделе **зонды** **\<resource-group-name> JSON** -файла:

        ```json
        "probes": [
                    {
                        "name": "myHTTPProbe",
                        "etag": "W/\"39e5e9cd-2d6d-491f-83cf-b37a259d86b6\"",
                        "properties": {
                            "provisioningState": "Succeeded",
                            "protocol": "Http",
                            "port": 80,
                            "requestPath": "/",
                            "intervalInSeconds": 15,
                            "numberOfProbes": 2
                        }
                    }
                ],
        ```
       Дополнительные сведения о Azure Load Balancer зондах работоспособности см. в разделе [проверки работоспособности Load Balancer](./load-balancer-custom-probe-overview.md) .

    * **Правила NAT для входящего трафика** . Вы можете добавить или удалить правила NAT для входящего трафика для подсистемы балансировки нагрузки, добавив или удалив записи в разделе **inboundNatRules** **\<resource-group-name> JSON** -файла:

        ```json
        "inboundNatRules": [
                    {
                        "name": "myInboundNATRule",
                        "etag": "W/\"39e5e9cd-2d6d-491f-83cf-b37a259d86b6\"",
                        "properties": {
                            "provisioningState": "Succeeded",
                            "frontendIPConfiguration": {
                                "id": "[concat(resourceId('Microsoft.Network/loadBalancers', parameters('loadBalancers_myLoadBalancer_name')), '/frontendIPConfigurations/myfrontendIPinbound')]"
                            },
                            "frontendPort": 4422,
                            "backendPort": 3389,
                            "enableFloatingIP": false,
                            "idleTimeoutInMinutes": 4,
                            "protocol": "Tcp",
                            "enableTcpReset": false
                        }
                    }
                ]
        ```
        Чтобы завершить добавление или удаление правила NAT для входящего трафика, оно должно присутствовать или удалено в качестве свойства **типа** в конце **\<resource-group-name> JSON** -файла:

        ```json
        {
            "type": "Microsoft.Network/loadBalancers/inboundNatRules",
            "apiVersion": "2019-06-01",
            "name": "[concat(parameters('loadBalancers_myLoadBalancer_name'), '/myInboundNATRule')]",
            "dependsOn": [
                "[resourceId('Microsoft.Network/loadBalancers', parameters('loadBalancers_myLoadBalancer_name'))]"
            ],
            "properties": {
                "provisioningState": "Succeeded",
                "frontendIPConfiguration": {
                    "id": "[concat(resourceId('Microsoft.Network/loadBalancers', parameters('loadBalancers_myLoadBalancer_name')), '/frontendIPConfigurations/myfrontendIPinbound')]"
                },
                "frontendPort": 4422,
                "backendPort": 3389,
                "enableFloatingIP": false,
                "idleTimeoutInMinutes": 4,
                "protocol": "Tcp",
                "enableTcpReset": false
            }
        }
        ```
        Дополнительные сведения о правилах NAT для входящего трафика см. в разделе [что такое Azure Load Balancer?](./load-balancer-overview.md)

    * **Правила исходящего трафика** . Вы можете добавить или удалить правила исходящего трафика в конфигурации, изменив свойство **аутбаундрулес** в **\<resource-group-name> JSON** -файле:

        ```json
        "outboundRules": [
                    {
                        "name": "myOutboundRule",
                        "etag": "W/\"39e5e9cd-2d6d-491f-83cf-b37a259d86b6\"",
                        "properties": {
                            "provisioningState": "Succeeded",
                            "allocatedOutboundPorts": 10000,
                            "protocol": "All",
                            "enableTcpReset": false,
                            "idleTimeoutInMinutes": 15,
                            "backendAddressPool": {
                                "id": "[concat(resourceId('Microsoft.Network/loadBalancers', parameters('loadBalancers_myLoadBalancer_name')), '/backendAddressPools/myBEPoolOutbound')]"
                            },
                            "frontendIPConfigurations": [
                                {
                                    "id": "[concat(resourceId('Microsoft.Network/loadBalancers', parameters('loadBalancers_myLoadBalancer_name')), '/frontendIPConfigurations/myfrontendIPoutbound')]"
                                }
                            ]
                        }
                    }
                ]
        ```

         Дополнительные сведения о правилах исходящих подключений см. в разделе [Load Balancer правила исходящего трафика](./load-balancer-outbound-connections.md#outboundrules) .

13. Сохраните файл **\<resource-group-name>.json**.
    
10. Создайте или группу ресурсов в целевом регионе для развертывания целевой внешней подсистемы балансировки нагрузки с помощью [New-азресаурцеграуп](/powershell/module/az.resources/new-azresourcegroup). Существующую группу ресурсов, приведенную выше, можно также использовать повторно в рамках этого процесса:
    
    ```azurepowershell-interactive
    New-AzResourceGroup -Name <target-resource-group-name> -location <target-region>
    ```
11. Разверните измененный файл **\<resource-group-name>.json** в группе ресурсов, созданной на предыдущем шаге, с помощью команды [New-AzResourceGroupDeployment](/powershell/module/az.resources/new-azresourcegroupdeployment):

    ```azurepowershell-interactive

    New-AzResourceGroupDeployment -ResourceGroupName <target-resource-group-name> -TemplateFile <source-resource-group-name>.json
    
    ```

12. Чтобы проверить, созданы ли ресурсы в целевом регионе, используйте [Get-азресаурцеграуп](/powershell/module/az.resources/get-azresourcegroup) и [Get-азлоадбаланцер](/powershell/module/az.network/get-azloadbalancer):
    
    ```azurepowershell-interactive

    Get-AzResourceGroup -Name <target-resource-group-name>

    ```

    ```azurepowershell-interactive

    Get-AzLoadBalancer -Name <target-publicip-name> -ResourceGroupName <target-resource-group-name>

    ```

## <a name="discard"></a>Игнорировать 

Если после развертывания вы хотите начать или отклонить общедоступный IP-адрес и подсистему балансировки нагрузки в целевом объекте, удалите группу ресурсов, созданную в целевом объекте, и перемещенный общедоступный IP-адрес и подсистема балансировки нагрузки будут удалены.  Чтобы удалить группу ресурсов, используйте команду [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup):

```azurepowershell-interactive

Remove-AzResourceGroup -Name <resource-group-name>

```

## <a name="clean-up"></a>Очистка

Чтобы зафиксировать изменения и завершить перемещение NSG, удалите исходный NSG или группу ресурсов, используйте [Remove-азресаурцеграуп](/powershell/module/az.resources/remove-azresourcegroup) или Remove- [АзпублиЦипаддресс](/powershell/module/az.network/remove-azpublicipaddress) и [Remove-азлоадбаланцер](/powershell/module/az.network/remove-azloadbalancer) .

```azurepowershell-interactive

Remove-AzResourceGroup -Name <resource-group-name>

```

``` azurepowershell-interactive

Remove-AzLoadBalancer -name <load-balancer> -ResourceGroupName <resource-group-name>

Remove-AzPublicIpAddress -Name <public-ip> -ResourceGroupName <resource-group-name>


```

## <a name="next-steps"></a>Дальнейшие действия

В этом руководстве вы переместили группу безопасности сети Azure из одного региона в другой и очистили исходные ресурсы.  Дополнительные сведения о перемещении ресурсов между регионами и аварийном восстановлении в Azure см. по следующей ссылке:


- [Перемещение ресурсов в новую группу ресурсов или подписку](../azure-resource-manager/management/move-resource-group-and-subscription.md)
- [Перенос виртуальных машин Azure в другой регион](../site-recovery/azure-to-azure-tutorial-migrate.md)