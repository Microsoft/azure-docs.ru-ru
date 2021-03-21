---
title: Перемещение группы безопасности сети Azure (NSG) в другой регион Azure с помощью Azure PowerShell
description: Используйте шаблон Azure Resource Manager, чтобы переместить группу безопасности сети Azure из одного региона Azure в другой с помощью Azure PowerShell.
author: asudbring
ms.service: virtual-network
ms.topic: how-to
ms.date: 08/31/2019
ms.author: allensu
ms.openlocfilehash: ad73ef03aa9623fb724f1397697fac18f659a90c
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98934994"
---
# <a name="move-azure-network-security-group-nsg-to-another-region-using-azure-powershell"></a>Перемещение группы безопасности сети Azure (NSG) в другой регион с помощью Azure PowerShell

Существуют различные сценарии, в которых необходимо переместить существующий группы безопасности сети из одного региона в другой. Например, может потребоваться создать NSG с теми же конфигурацией и правилами безопасности для тестирования. Вы также можете переместить NSG в другой регион в рамках планирования аварийного восстановления.

Группы безопасности Azure нельзя перемещать из одного региона в другой. Однако можно использовать шаблон Azure Resource Manager для экспорта существующих правил конфигурации и безопасности NSG.  Затем можно разместить ресурс в другом регионе, экспортировав NSG в шаблон, изменив параметры в соответствии с регионом назначения, а затем развернув шаблон в новом регионе.  Дополнительные сведения о диспетчер ресурсов и шаблонах см. [в разделе Экспорт групп ресурсов в шаблоны](../azure-resource-manager/management/manage-resource-groups-powershell.md#export-resource-groups-to-templates).


## <a name="prerequisites"></a>Предварительные условия

- Убедитесь, что группа безопасности сети Azure находится в регионе Azure, из которого вы хотите переместиться.

- Группы безопасности сети Azure нельзя перемещать между регионами.  Необходимо связать новый NSG с ресурсами в целевом регионе.

- Чтобы экспортировать конфигурацию NSG и развернуть шаблон для создания NSG в другом регионе, вам потребуется роль «участник сети» или выше.
   
- Определите структуру сети в исходном регионе и все ресурсы, которые вы сейчас используете. Этот макет включает, но не ограничен подсистемами балансировки нагрузки, общедоступными IP-адресами и виртуальными сетями.

- Убедитесь, что ваша подписка Azure позволяет создать группы безопасности сети в используемом целевом регионе. Свяжитесь со службой поддержки, чтобы включить необходимые квоты.

- Убедитесь, что у вашей подписки достаточно ресурсов для поддержки добавления группы безопасности сети для этого процесса.  Ознакомьтесь со статьей [Подписка Azure, границы, квоты и ограничения службы](../azure-resource-manager/management/azure-subscription-service-limits.md#networking-limits).


## <a name="prepare-and-move"></a>Подготовка и перемещение
Ниже показано, как подготовить группу безопасности сети для переноса правила конфигурации и безопасности с помощью шаблона диспетчер ресурсов и переместить конфигурацию NSG и правила безопасности в целевой регион с помощью Azure PowerShell.


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

### <a name="export-the-template-and-deploy-from-a-script"></a>Экспорт шаблона и развертывание из скрипта

1. С помощью команды [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount) войдите в подписку Azure и следуйте инструкциям на экране:
    
    ```azurepowershell-interactive
    Connect-AzAccount
    ```

2. Получите идентификатор ресурса NSG, который необходимо переместить в целевой регион, и поместите его в переменную с помощью [Get-азнетворксекуритиграуп](/powershell/module/az.network/get-aznetworksecuritygroup):

    ```azurepowershell-interactive
    $sourceNSGID = (Get-AzNetworkSecurityGroup -Name <source-nsg-name> -ResourceGroupName <source-resource-group-name>).Id

    ```
3. Экспортируйте исходный NSG в JSON-файл в каталог, в котором выполняется команда [Export — азресаурцеграуп](/powershell/module/az.resources/export-azresourcegroup):
   
   ```azurepowershell-interactive
   Export-AzResourceGroup -ResourceGroupName <source-resource-group-name> -Resource $sourceNSGID -IncludeParameterDefaultValue
   ```

4. Скачанный файл будет называться именем группы ресурсов, из которой был экспортирован ресурс.  Выберите файл, экспортированный из команды, с именем **\<resource-group-name>.json**, и откройте его в любом редакторе:
   
   ```azurepowershell
   notepad <source-resource-group-name>.json
   ```

5. Чтобы изменить параметр имени NSG, измените значение **DefaultValue** для свойства Source NSG Name на имя целевой NSG, а затем убедитесь, что оно находится в кавычках:
    
    ```json
            {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
            "networkSecurityGroups_myVM1_nsg_name": {
            "defaultValue": "<target-nsg-name>",
            "type": "String"
            }
        }

    ```


6. Чтобы изменить целевой регион, в который будут перемещены конфигурация NSG и правила безопасности, измените свойство **Location** в разделе **ресурсы**:

    ```json
            "resources": [
            {
            "type": "Microsoft.Network/networkSecurityGroups",
            "apiVersion": "2019-06-01",
            "name": "[parameters('networkSecurityGroups_myVM1_nsg_name')]",
            "location": "<target-region>",
            "properties": {
                "provisioningState": "Succeeded",
                "resourceGuid": "2c846acf-58c8-416d-be97-ccd00a4ccd78", 
             }
            }
    ```
  
7. Чтобы получить коды расположения регионов, можно использовать командлет Azure PowerShell [Get-AzLocation](/powershell/module/az.resources/get-azlocation), выполнив следующую команду:

    ```azurepowershell-interactive

    Get-AzLocation | format-table
    
    ```
8. Кроме того, можно изменить другие параметры в **\<resource-group-name> JSON** , если выбрать, и они необязательны в зависимости от требований:

    * **Правила безопасности** . правила, которые развертываются в целевом NSG, можно изменить путем добавления или удаления правил в раздел **securityRules** в **\<resource-group-name> JSON** -файле:

        ```json
           "resources": [
                  {
                  "type": "Microsoft.Network/networkSecurityGroups",
                  "apiVersion": "2019-06-01",
                  "name": "[parameters('networkSecurityGroups_myVM1_nsg_name')]",
                  "location": "TARGET REGION",
                  "properties": {
                       "provisioningState": "Succeeded",
                       "resourceGuid": "2c846acf-58c8-416d-be97-ccd00a4ccd78",
                  "securityRules": [
                    {
                        "name": "RDP",
                        "etag": "W/\"c630c458-6b52-4202-8fd7-172b7ab49cf5\"",
                        "properties": {
                             "provisioningState": "Succeeded",
                             "protocol": "TCP",
                             "sourcePortRange": "*",
                             "destinationPortRange": "3389",
                             "sourceAddressPrefix": "*",
                             "destinationAddressPrefix": "*",
                             "access": "Allow",
                             "priority": 300,
                             "direction": "Inbound",
                             "sourcePortRanges": [],
                             "destinationPortRanges": [],
                             "sourceAddressPrefixes": [],
                             "destinationAddressPrefixes": []
                            }
                        ]
            }  
            
        ```

        Чтобы завершить добавление или удаление правил в целевом NSG, необходимо также изменить типы настраиваемых правил в конце **\<resource-group-name> JSON** – файла в формате, приведенном ниже.

        ```json
           {
            "type": "Microsoft.Network/networkSecurityGroups/securityRules",
            "apiVersion": "2019-06-01",
            "name": "[concat(parameters('networkSecurityGroups_myVM1_nsg_name'), '/Port_80')]",
            "dependsOn": [
                "[resourceId('Microsoft.Network/networkSecurityGroups', parameters('networkSecurityGroups_myVM1_nsg_name'))]"
            ],
            "properties": {
                "provisioningState": "Succeeded",
                "protocol": "*",
                "sourcePortRange": "*",
                "destinationPortRange": "80",
                "sourceAddressPrefix": "*",
                "destinationAddressPrefix": "*",
                "access": "Allow",
                "priority": 310,
                "direction": "Inbound",
                "sourcePortRanges": [],
                "destinationPortRanges": [],
                "sourceAddressPrefixes": [],
                "destinationAddressPrefixes": []
            }
        ```

9. Сохраните файл **\<resource-group-name>.json**.

10. Создайте группу ресурсов в целевом регионе для развертывания целевого NSG с помощью [New-азресаурцеграуп](/powershell/module/az.resources/new-azresourcegroup):
    
    ```azurepowershell-interactive
    New-AzResourceGroup -Name <target-resource-group-name> -location <target-region>
    ```
    
11. Разверните измененный файл **\<resource-group-name>.json** в группе ресурсов, созданной на предыдущем шаге, с помощью команды [New-AzResourceGroupDeployment](/powershell/module/az.resources/new-azresourcegroupdeployment):

    ```azurepowershell-interactive

    New-AzResourceGroupDeployment -ResourceGroupName <target-resource-group-name> -TemplateFile <source-resource-group-name>.json
    
    ```

12. Чтобы проверить, созданы ли ресурсы в целевом регионе, используйте [Get-азресаурцеграуп](/powershell/module/az.resources/get-azresourcegroup) и [Get-азнетворксекуритиграуп](/powershell/module/az.network/get-aznetworksecuritygroup):
    
    ```azurepowershell-interactive

    Get-AzResourceGroup -Name <target-resource-group-name>

    ```

    ```azurepowershell-interactive

    Get-AzNetworkSecurityGroup -Name <target-nsg-name> -ResourceGroupName <target-resource-group-name>

    ```

## <a name="discard"></a>Игнорировать 

Если после развертывания вы хотите начать или отклонить NSG в целевом объекте, удалите группу ресурсов, созданную в целевом объекте, и перемещенный NSG будет удален.  Чтобы удалить группу ресурсов, используйте команду [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup):

```azurepowershell-interactive

Remove-AzResourceGroup -Name <target-resource-group-name>

```

## <a name="clean-up"></a>Очистка

Чтобы сохранить изменения и завершить перемещение NSG, удалите исходный NSG или группу ресурсов, используйте [Remove-азресаурцеграуп](/powershell/module/az.resources/remove-azresourcegroup) или [Remove-азнетворксекуритиграуп](/powershell/module/az.network/remove-aznetworksecuritygroup):

```azurepowershell-interactive

Remove-AzResourceGroup -Name <source-resource-group-name>

```

``` azurepowershell-interactive

Remove-AzNetworkSecurityGroup -Name <source-nsg-name> -ResourceGroupName <source-resource-group-name>

```

## <a name="next-steps"></a>Дальнейшие действия

В этом руководстве вы переместили группу безопасности сети Azure из одного региона в другой и очистили исходные ресурсы.  Дополнительные сведения о перемещении ресурсов между регионами и аварийном восстановлении в Azure см. по следующей ссылке:


- [Перемещение ресурсов в новую группу ресурсов или подписку](../azure-resource-manager/management/move-resource-group-and-subscription.md)
- [Перенос виртуальных машин Azure в другой регион](../site-recovery/azure-to-azure-tutorial-migrate.md)
