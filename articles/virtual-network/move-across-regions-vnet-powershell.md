---
title: Перемещение виртуальной сети Azure в другой регион Azure с помощью Azure PowerShell
description: Перемещение виртуальной сети Azure из одного региона Azure в другой с помощью шаблона диспетчер ресурсов и Azure PowerShell.
author: asudbring
ms.service: virtual-network
ms.topic: how-to
ms.date: 08/26/2019
ms.author: allensu
ms.openlocfilehash: fa89a458b23d18bc6fddfca9cf6d9f2a16f91669
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98934928"
---
# <a name="move-an-azure-virtual-network-to-another-region-by-using-azure-powershell"></a>Перемещение виртуальной сети Azure в другой регион с помощью Azure PowerShell

Существует несколько сценариев перемещения существующей виртуальной сети Azure из одного региона в другой. Например, может потребоваться создать виртуальную сеть с такой же конфигурацией для тестирования и доступности, что и у существующей виртуальной сети. Кроме того, вы можете переместить производственную виртуальную сеть в другой регион в рамках планирования аварийного восстановления.

Вы можете использовать шаблон Azure Resource Manager для завершения перемещения виртуальной сети в другой регион. Для этого необходимо экспортировать виртуальную сеть в шаблон, изменить параметры в соответствии с регионом назначения, а затем развернуть шаблон в новом регионе. Дополнительные сведения о шаблонах диспетчер ресурсов см. [в разделе Экспорт групп ресурсов в шаблоны](../azure-resource-manager/management/manage-resource-groups-powershell.md#export-resource-groups-to-templates).


## <a name="prerequisites"></a>Предварительные условия

- Убедитесь, что виртуальная сеть находится в регионе Azure, из которого вы хотите переместиться.

- Чтобы экспортировать виртуальную сеть и развернуть шаблон для создания виртуальной сети в другом регионе, необходимо иметь роль участника сети или более высокий.

- Пиринг виртуальных сетей не будет создан повторно, и они завершатся ошибкой, если они все еще находятся в шаблоне. Перед экспортом шаблона необходимо удалить все узлы виртуальных сетей. Затем их можно будет восстановить после перемещения виртуальной сети.
    
- Определите структуру сети в исходном регионе и все ресурсы, которые вы сейчас используете. Этот макет включает, но не ограничен подсистемами балансировки нагрузки, группами безопасности сети (группы безопасности сети) и общедоступными IP-адресами.

- Убедитесь, что подписка Azure позволяет создавать виртуальные сети в целевом регионе. Чтобы включить требуемую квоту, обратитесь в службу поддержки.

- Убедитесь, что у вашей подписки достаточно ресурсов для поддержки добавления виртуальных сетей для этого процесса. Дополнительные сведения см. в статье [Подписка Azure, границы, квоты и ограничения службы](../azure-resource-manager/management/azure-subscription-service-limits.md#networking-limits).


## <a name="prepare-for-the-move"></a>Подготовка к перемещению
В этом разделе вы подготовите виртуальную сеть для перемещения с помощью шаблона диспетчер ресурсов. Затем вы перемещаете виртуальную сеть в целевой регион с помощью команд Azure PowerShell.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Чтобы экспортировать виртуальную сеть и развернуть целевую виртуальную сеть с помощью PowerShell, выполните следующие действия.

1. Войдите в подписку Azure с помощью команды [Connect-азаккаунт](/powershell/module/az.accounts/connect-azaccount) , а затем следуйте инструкциям на экране.
    
    ```azurepowershell-interactive
    Connect-AzAccount
    ```

1. Получите идентификатор ресурса виртуальной сети, которую необходимо переместить в целевой регион, а затем поместите ее в переменную с помощью команды [Get-азвиртуалнетворк](/powershell/module/az.network/get-azvirtualnetwork):

    ```azurepowershell-interactive
    $sourceVNETID = (Get-AzVirtualNetwork -Name <source-virtual-network-name> -ResourceGroupName <source-resource-group-name>).Id
    ```

1. Экспортируйте исходную виртуальную сеть в файл JSON в каталоге, в котором выполняется команда [Export-азресаурцеграуп](/powershell/module/az.resources/export-azresourcegroup):
   
   ```azurepowershell-interactive
   Export-AzResourceGroup -ResourceGroupName <source-resource-group-name> -Resource $sourceVNETID -IncludeParameterDefaultValue
   ```

1. Имя скачанного файла совпадает с именем группы ресурсов, из которой был экспортирован ресурс. Найдите *\<resource-group-name> JSON* файл, экспортированный с помощью команды, и откройте его в редакторе:
   
   ```azurepowershell
   notepad <source-resource-group-name>.json
   ```

1. Чтобы изменить параметр имени виртуальной сети, измените свойство **DefaultValue** исходного имени виртуальной сети на имя целевой виртуальной сети. Не забудьте заключить имя в кавычки.
    
    ```json
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentmyResourceGroupVNET.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "virtualNetworks_myVNET1_name": {
        "defaultValue": "<target-virtual-network-name>",
        "type": "String"
        }
    ```

1. Чтобы изменить целевой регион, в который будет перемещена виртуальная сеть, измените свойство **Location** в разделе ресурсы:

    ```json
    "resources": [
                {
                    "type": "Microsoft.Network/virtualNetworks",
                    "apiVersion": "2019-06-01",
                    "name": "[parameters('virtualNetworks_myVNET1_name')]",
                    "location": "<target-region>",
                    "properties": {
                        "provisioningState": "Succeeded",
                        "resourceGuid": "6e2652be-35ac-4e68-8c70-621b9ec87dcb",
                        "addressSpace": {
                            "addressPrefixes": [
                                "10.0.0.0/16"
                            ]
                        },

    ```
  
1. Чтобы получить коды расположения регионов, можно использовать командлет Azure PowerShell [Get-AzLocation](/powershell/module/az.resources/get-azlocation), выполнив следующую команду:

    ```azurepowershell-interactive

    Get-AzLocation | format-table
    ```

1. Используемых В зависимости от требований можно также изменить другие параметры в файле *\<resource-group-name> JSON* .

    * **Адресное пространство**. перед сохранением файла можно изменить адресное пространство виртуальной сети, изменив раздел **Resources**  >  **аддрессспаце** и изменив свойство **аддресспрефиксес** :

        ```json
                "resources": [
                    {
                    "type": "Microsoft.Network/virtualNetworks",
                    "apiVersion": "2019-06-01",
                    "name": "[parameters('virtualNetworks_myVNET1_name')]",
                    "location": "<target-region",
                    "properties": {
                    "provisioningState": "Succeeded",
                    "resourceGuid": "6e2652be-35ac-4e68-8c70-621b9ec87dcb",
                    "addressSpace": {
                        "addressPrefixes": [
                        "10.0.0.0/16"
                        ]
                    },
        ```

    * **Подсеть**. можно изменить или добавить имя подсети и адресное пространство подсети, изменив раздел **подсети** файла. Имя подсети можно изменить, изменив свойство **Name** . Адресное пространство подсети можно изменить, изменив свойство **addressPrefix** :

        ```json
                "subnets": [
                    {
                    "name": "subnet-1",
                    "etag": "W/\"d9f6e6d6-2c15-4f7c-b01f-bed40f748dea\"",
                    "properties": {
                    "provisioningState": "Succeeded",
                    "addressPrefix": "10.0.0.0/24",
                    "delegations": [],
                    "privateEndpointNetworkPolicies": "Enabled",
                    "privateLinkServiceNetworkPolicies": "Enabled"
                    }
                    },
                    {
                    "name": "GatewaySubnet",
                    "etag": "W/\"d9f6e6d6-2c15-4f7c-b01f-bed40f748dea\"",
                    "properties": {
                    "provisioningState": "Succeeded",
                    "addressPrefix": "10.0.1.0/29",
                    "serviceEndpoints": [],
                    "delegations": [],
                    "privateEndpointNetworkPolicies": "Enabled",
                    "privateLinkServiceNetworkPolicies": "Enabled"
                    }
                    }

                ]
        ```

        Чтобы изменить префикс адреса, измените файл в двух местах: в коде, приведенном в предыдущем разделе, и в разделе **Type** следующего кода. Измените свойство **addressPrefix** в следующем коде, чтобы оно соответствовало свойству **addressPrefix** в коде в предыдущем разделе.

        ```json
         "type": "Microsoft.Network/virtualNetworks/subnets",
           "apiVersion": "2019-06-01",
           "name": "[concat(parameters('virtualNetworks_myVNET1_name'), '/GatewaySubnet')]",
              "dependsOn": [
                 "[resourceId('Microsoft.Network/virtualNetworks', parameters('virtualNetworks_myVNET1_name'))]"
                   ],
              "properties": {
                 "provisioningState": "Succeeded",
                 "addressPrefix": "10.0.1.0/29",
                 "serviceEndpoints": [],
                 "delegations": [],
                 "privateEndpointNetworkPolicies": "Enabled",
                 "privateLinkServiceNetworkPolicies": "Enabled"
                  }
                 },
                  {
                  "type": "Microsoft.Network/virtualNetworks/subnets",
                  "apiVersion": "2019-06-01",
                  "name": "[concat(parameters('virtualNetworks_myVNET1_name'), '/subnet-1')]",
                     "dependsOn": [
                        "[resourceId('Microsoft.Network/virtualNetworks', parameters('virtualNetworks_myVNET1_name'))]"
                          ],
                     "properties": {
                        "provisioningState": "Succeeded",
                        "addressPrefix": "10.0.0.0/24",
                        "delegations": [],
                        "privateEndpointNetworkPolicies": "Enabled",
                        "privateLinkServiceNetworkPolicies": "Enabled"
                         }
                  }
         ]
        ```

1. Сохраните файл *\<resource-group-name>.json*.

1. Создайте группу ресурсов в целевом регионе для развертывания целевой виртуальной сети с помощью команды [New-азресаурцеграуп](/powershell/module/az.resources/new-azresourcegroup):
    
    ```azurepowershell-interactive
    New-AzResourceGroup -Name <target-resource-group-name> -location <target-region>
    ```
    
1. Разверните отредактированный *\<resource-group-name> JSON* -файл в группе ресурсов, созданной на предыдущем шаге, с помощью команды [New-азресаурцеграупдеплоймент](/powershell/module/az.resources/new-azresourcegroupdeployment):

    ```azurepowershell-interactive

    New-AzResourceGroupDeployment -ResourceGroupName <target-resource-group-name> -TemplateFile <source-resource-group-name>.json
    ```

1. Чтобы убедиться, что ресурсы были созданы в целевом регионе, используйте [Get-азресаурцеграуп](/powershell/module/az.resources/get-azresourcegroup) и [Get-азвиртуалнетворк](/powershell/module/az.network/get-azvirtualnetwork):
    
    ```azurepowershell-interactive

    Get-AzResourceGroup -Name <target-resource-group-name>
    ```

    ```azurepowershell-interactive

    Get-AzVirtualNetwork -Name <target-virtual-network-name> -ResourceGroupName <target-resource-group-name>
    ```

## <a name="delete-the-virtual-network-or-resource-group"></a>Удаление виртуальной сети или группы ресурсов 

После развертывания виртуальной сети для запуска или удаления виртуальной сети в целевом регионе удалите группу ресурсов, созданную в целевом регионе, и перемещенная виртуальная сеть будет удалена. 

Чтобы удалить группу ресурсов, используйте команду [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup):

```azurepowershell-interactive

Remove-AzResourceGroup -Name <target-resource-group-name>
```

## <a name="clean-up"></a>Очистка

Чтобы зафиксировать изменения и завершить перемещение виртуальной сети, выполните одно из следующих действий.

* Удалите группу ресурсов с помощью команды [Remove-азресаурцеграуп](/powershell/module/az.resources/remove-azresourcegroup):

    ```azurepowershell-interactive

    Remove-AzResourceGroup -Name <source-resource-group-name>
    ```

* Удалите исходную виртуальную сеть с помощью [Remove-азвиртуалнетворк](/powershell/module/az.network/remove-azvirtualnetwork):  
    ``` azurepowershell-interactive

    Remove-AzVirtualNetwork -Name <source-virtual-network-name> -ResourceGroupName <source-resource-group-name>
    ```

## <a name="next-steps"></a>Дальнейшие действия

В этом руководстве вы переместили виртуальную сеть из одного региона в другой с помощью PowerShell, а затем очистили ненужные исходные ресурсы. Дополнительные сведения о перемещении ресурсов между регионами и аварийным восстановлением в Azure см. в следующих статьях:

- [Перемещение ресурсов в новую группу ресурсов или подписку](../azure-resource-manager/management/move-resource-group-and-subscription.md)
- [Перемещение виртуальных машин Azure в другой регион](../site-recovery/azure-to-azure-tutorial-migrate.md)
