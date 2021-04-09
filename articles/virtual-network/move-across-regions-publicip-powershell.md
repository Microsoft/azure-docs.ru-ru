---
title: Перемещение конфигурации общедоступного IP-адреса Azure в другой регион Azure с помощью Azure PowerShell
description: Используйте шаблон Azure Resource Manager, чтобы переместить конфигурацию общедоступного IP-адреса Azure из одного региона Azure в другой с помощью Azure PowerShell.
author: asudbring
ms.service: virtual-network
ms.subservice: ip-services
ms.topic: how-to
ms.date: 08/29/2019
ms.author: allensu
ms.openlocfilehash: a21d088680855b74e7259028ed7ef55165707c56
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98938697"
---
# <a name="move-azure-public-ip-configuration-to-another-region-using-azure-powershell"></a>Перемещение конфигурации общедоступного IP-адреса Azure в другой регион с помощью Azure PowerShell

Существуют разные причины, по которым может потребоваться перенос существующих конфигураций общедоступного IP-адреса Azure из одного региона в другой. Например, вы хотите создать общедоступный IP-адрес с такой же конфигурацией и SKU для тестирования. Или вы перемещаете конфигурацию в рамках планирования аварийного восстановления.

**Общедоступные IP-адреса Azure относятся к регионам и не могут перемещаться из одного региона в другой.** Однако можно использовать шаблон Azure Resource Manager для экспорта существующей конфигурации общедоступного IP-адреса.  Затем можно разместить ресурс в другом регионе, экспортировав общедоступный IP-адрес в шаблон, изменив параметры в соответствии с регионом назначения и развернув шаблон в новом регионе.  Дополнительные сведения о Resource Manager и шаблонах см. в разделе [Экспорт групп ресурсов в шаблоны](../azure-resource-manager/management/manage-resource-groups-powershell.md#export-resource-groups-to-templates)


## <a name="prerequisites"></a>Предварительные требования

- Убедитесь, что общедоступный IP-адрес Azure расположен в регионе Azure, из которого вы намерены его перенести.

- Общедоступные IP-адреса Azure нельзя перемещать между регионами.  Необходимо связать новый общедоступный IP-адрес с ресурсами в целевом регионе.

- Чтобы экспортировать конфигурацию общедоступного IP-адреса и развернуть шаблон для создания общедоступного IP-адреса в другом регионе, вам потребуется роль "Участник сети" или выше.
   
- Определите структуру сети в исходном регионе и все ресурсы, которые вы сейчас используете. Сюда могут входить, среди прочего, подсистемы балансировки нагрузки, группы безопасности сети и виртуальные сети.

- Убедитесь, что используемая подписка Azure позволяет создавать общедоступные IP-адреса в целевом регионе. Свяжитесь со службой поддержки, чтобы включить необходимые квоты.

- Убедитесь, что у вашей подписки достаточно ресурсов для поддержки добавления общедоступных IP-адресов для этого процесса.  Ознакомьтесь со статьей [Подписка Azure, границы, квоты и ограничения службы](../azure-resource-manager/management/azure-subscription-service-limits.md#networking-limits).


## <a name="prepare-and-move"></a>Подготовка и перемещение
Ниже описано, как подготовить общедоступный IP-адрес для перемещения конфигурации с помощью шаблона Resource Manager и переместить конфигурацию общедоступного IP-адреса в целевой регион с помощью Azure PowerShell.


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

### <a name="export-the-template-and-deploy-from-a-script"></a>Экспорт шаблона и развертывание из скрипта

1. С помощью команды [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount) войдите в подписку Azure и следуйте инструкциям на экране:
    
    ```azurepowershell-interactive
    Connect-AzAccount
    ```

2. Получите идентификатор ресурса общедоступного IP-адреса, который необходимо переместить в целевой регион, и включите его в переменную с помощью команды [Get-AzPublicIPAddress](/powershell/module/az.network/get-azpublicipaddress):

    ```azurepowershell-interactive
    $sourcePubIPID = (Get-AzPublicIPaddress -Name <source-public-ip-name> -ResourceGroupName <source-resource-group-name>).Id

    ```
3. Экспортируйте исходную виртуальную сеть в файл JSON в каталог, в котором выполняется команда [Export-AzResourceGroup](/powershell/module/az.resources/export-azresourcegroup):
   
   ```azurepowershell-interactive
   Export-AzResourceGroup -ResourceGroupName <source-resource-group-name> -Resource $sourceVNETID -IncludeParameterDefaultValue
   ```

4. Скачанный файл будет называться именем группы ресурсов, из которой был экспортирован ресурс.  Выберите файл, экспортированный из команды, с именем **\<resource-group-name>.json**, и откройте его в любом редакторе:
   
   ```azurepowershell
   notepad <source-resource-group-name>.json
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

         Дополнительные сведения о различиях между общедоступными IP-адресами уровня "Базовый" и "Стандартный" см. в разделе [Создание, изменение или удаление общедоступного IP-адреса](./virtual-network-public-ip-address.md).

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

        Дополнительные сведения о методах выделения и значениях времени ожидания при простое см. в разделе [Создание, изменение или удаление общедоступного IP-адреса](./virtual-network-public-ip-address.md).


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
## <a name="discard"></a>Игнорировать 

Если после развертывания вы хотите начать заново или отклонить общедоступный IP-адрес в целевом объекте, удалите группу ресурсов, созданную в целевом объекте, и перемещенный общедоступный IP-адрес будет удален.  Чтобы удалить группу ресурсов, используйте команду [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup):

```azurepowershell-interactive

Remove-AzResourceGroup -Name <target-resource-group-name>

```

## <a name="clean-up"></a>Очистка

Чтобы зафиксировать изменения и завершить перемещение виртуальной сети, удалите исходную виртуальную сеть или группу ресурсов, используя [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) или [Remove-AzPublicIPAddress](/powershell/module/az.network/remove-azpublicipaddress):

```azurepowershell-interactive

Remove-AzResourceGroup -Name <source-resource-group-name>

```

``` azurepowershell-interactive

Remove-AzPublicIpAddress -Name <source-publicip-name> -ResourceGroupName <resource-group-name>

```

## <a name="next-steps"></a>Дальнейшие действия

В этом руководстве вы переместили общедоступный IP-адрес Azure из одного региона в другой и очистили исходные ресурсы.  Дополнительные сведения о перемещении ресурсов между регионами и аварийном восстановлении в Azure см. по следующей ссылке:


- [Перемещение ресурсов в новую группу ресурсов или подписку](../azure-resource-manager/management/move-resource-group-and-subscription.md)
- [Перенос виртуальных машин Azure в другой регион](../site-recovery/azure-to-azure-tutorial-migrate.md)
