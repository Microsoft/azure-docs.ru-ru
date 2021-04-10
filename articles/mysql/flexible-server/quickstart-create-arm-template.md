---
title: Краткое руководство. Создание Базы данных Azure для MySQL (Гибкий сервер) с помощью шаблона ARM
description: Из этого краткого руководства вы узнаете, как создать Базу данных Azure для MySQL (Гибкий сервер) с помощью шаблона ARM.
author: mksuni
ms.service: mysql
ms.topic: quickstart
ms.custom: subject-armqs
ms.author: sumuth
ms.date: 10/23/2020
ms.openlocfilehash: def9e4f1b3f1c4e8f88f77dfe6906a8c96a94744
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "100389473"
---
# <a name="quickstart-use-an-arm-template-to-create-an-azure-database-for-mysql---flexible-server-preview"></a>Краткое руководство. Создание Базы данных Azure для MySQL (Гибкий сервер) (предварительная версия) с помощью шаблона ARM

> [!IMPORTANT]
> Сейчас предоставляется общедоступная предварительная версия Гибкого сервера Базы данных Azure для MySQL.

База данных Azure для MySQL (Гибкий сервер) (предварительная версия) — это управляемая служба, которая позволяет запускать, администрировать и масштабировать в облаке базы данных MySQL с высоким уровнем доступности. Шаблон Azure Resource Manager (ARM) можно использовать для подготовки гибкого сервера к работе, чтобы развернуть несколько серверов или несколько баз данных на сервере.

[!INCLUDE [About Azure Resource Manager](../../../includes/resource-manager-quickstart-introduction.md)]

## <a name="prerequisites"></a>Предварительные требования

Учетная запись Azure с активной подпиской. [Создайте бесплатно](https://azure.microsoft.com/free/).

## <a name="review-the-template"></a>Изучение шаблона

База данных Azure для MySQL (Гибкий сервер) выполняет функции родительского ресурса для одной или нескольких баз данных в пределах региона. Он обеспечивает область действия для политик управления, применяемых к базам данных: имена входа, брандмауэр, пользователи, роли и конфигурации.

Создайте файл _mysql-flexible-server-template.json_ и скопируйте в него этот скрипт JSON.

```json
{
  "$schema": "http://schema.management.azure.com/schemas/2014-04-01-preview/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "administratorLogin": {
      "type": "String"
    },
    "administratorLoginPassword": {
      "type": "SecureString"
    },
    "location": {
      "type": "String"
    },
    "serverName": {
      "type": "String"
    },
    "serverEdition": {
      "type": "String"
    },
    "storageSizeMB": {
      "type": "Int"
    },
    "haEnabled": {
      "type": "string",
      "defaultValue": "Disabled"
    },
    "availabilityZone": {
      "type": "String"
    },
    "version": {
      "type": "String"
    },
    "tags": {
      "defaultValue": {},
      "type": "Object"
    },
    "firewallRules": {
      "defaultValue": {},
      "type": "Object"
    },
    "vnetData": {
      "defaultValue": {},
      "type": "Object"
    },
    "backupRetentionDays": {
      "type": "Int"
    }
  },
  "variables": {
    "api": "2020-07-01-preview",
    "firewallRules": "[parameters('firewallRules').rules]",
    "publicNetworkAccess": "[if(empty(parameters('vnetData')), 'Enabled', 'Disabled')]",
    "vnetDataSet": "[if(empty(parameters('vnetData')), json('{ \"subnetArmResourceId\": \"\" }'), parameters('vnetData'))]",
    "finalVnetData": "[json(concat('{ \"subnetArmResourceId\": \"', variables('vnetDataSet').subnetArmResourceId, '\"}'))]"
  },
  "resources": [
    {
      "type": "Microsoft.DBforMySQL/flexibleServers",
      "apiVersion": "[variables('api')]",
      "name": "[parameters('serverName')]",
      "location": "[parameters('location')]",
      "sku": {
        "name": "Standard_D4ds_v4",
        "tier": "[parameters('serverEdition')]"
      },
      "tags": "[parameters('tags')]",
      "properties": {
        "version": "[parameters('version')]",
        "administratorLogin": "[parameters('administratorLogin')]",
        "administratorLoginPassword": "[parameters('administratorLoginPassword')]",
        "publicNetworkAccess": "[variables('publicNetworkAccess')]",
        "DelegatedSubnetArguments": "[if(empty(parameters('vnetData')), json('null'), variables('finalVnetData'))]",
        "haEnabled": "[parameters('haEnabled')]",
        "storageProfile": {
          "storageMB": "[parameters('storageSizeMB')]",
          "backupRetentionDays": "[parameters('backupRetentionDays')]"
        },
        "availabilityZone": "[parameters('availabilityZone')]"
      }
    },
    {
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2019-08-01",
      "name": "[concat('firewallRules-', copyIndex())]",
      "dependsOn": [
        "[concat('Microsoft.DBforMySQL/flexibleServers/', parameters('serverName'))]"
      ],
      "properties": {
        "mode": "Incremental",
        "template": {
          "$schema": "http://schema.management.azure.com/schemas/2014-04-01-preview/deploymentTemplate.json#",
          "contentVersion": "1.0.0.0",
          "resources": [
            {
              "type": "Microsoft.DBforMySQL/flexibleServers/firewallRules",
              "name": "[concat(parameters('serverName'),'/',variables('firewallRules')[copyIndex()].name)]",
              "apiVersion": "[variables('api')]",
              "properties": {
                "StartIpAddress": "[variables('firewallRules')[copyIndex()].startIPAddress]",
                "EndIpAddress": "[variables('firewallRules')[copyIndex()].endIPAddress]"
              }
            }
          ]
        }
      },
      "copy": {
        "name": "firewallRulesIterator",
        "count": "[if(greater(length(variables('firewallRules')), 0), length(variables('firewallRules')), 1)]",
        "mode": "Serial"
      },
      "condition": "[greater(length(variables('firewallRules')), 0)]"
    }
  ]
}
```

В шаблоне определены следующие ресурсы:

- Microsoft.DBforMySQL/flexibleServers

## <a name="deploy-the-template"></a>Развертывание шаблона

Выберите **Попробовать** в следующем блоке кода PowerShell, чтобы открыть [Azure Cloud Shell](../../cloud-shell/overview.md).

```azurepowershell-interactive
$serverName = Read-Host -Prompt "Enter a name for the new Azure Database for MySQL server"
$resourceGroupName = Read-Host -Prompt "Enter a name for the new resource group where the server will exist"
$location = Read-Host -Prompt "Enter an Azure region (for example, centralus) for the resource group"
$adminUser = Read-Host -Prompt "Enter the Azure Database for MySQL server's administrator account name"
$adminPassword = Read-Host -Prompt "Enter the administrator password" -AsSecureString

New-AzResourceGroup -Name $resourceGroupName -Location $location # Use this command when you need to create a new resource group for your deployment
New-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName `
    -TemplateFile "D:\Azure\Templates\EngineeringSite.json
    -serverName $serverName `
    -administratorLogin $adminUser `
    -administratorLoginPassword $adminPassword

Read-Host -Prompt "Press [ENTER] to continue ..."
```

## <a name="review-deployed-resources"></a>Просмотр развернутых ресурсов

Выполните следующие действия, чтобы проверить, создан ли сервер в Azure.

### <a name="azure-portal"></a>Портал Azure

1. На [портале Azure](https://portal.azure.com) выполните поиск по фразе **Серверы Базы данных Azure для MySQL** и выберите этот вариант.
1. В списке баз данных выберите новый сервер. Откроется страница **Обзор** для нового сервера Базы данных Azure для MySQL.

### <a name="powershell"></a>PowerShell

Чтобы просмотреть сведения о гибком сервере Базы данных Azure для MySQL, необходимо ввести имя нового сервера.

```azurepowershell-interactive
$serverName = Read-Host -Prompt "Enter the name of your Azure Database for MySQL server"
Get-AzResource -ResourceType "Microsoft.DBforMySQL/flexibleServers" -Name $serverName | ft
Write-Host "Press [ENTER] to continue..."
```

### <a name="cli"></a>CLI

Чтобы просмотреть сведения о гибком сервере Базы данных Azure для MySQL, необходимо ввести имя нового сервера и группу ресурсов.

```azurecli-interactive
echo "Enter your Azure Database for MySQL server name:" &&
read serverName &&
echo "Enter the resource group where the Azure Database for MySQL server exists:" &&
read resourcegroupName &&
az resource show --resource-group $resourcegroupName --name $serverName --resource-type "Microsoft.DbForMySQL/flexibleServers"
```

## <a name="clean-up-resources"></a>Очистка ресурсов

Сохраните эту группу ресурсов, сервер и отдельную базу данных, чтобы перейти к [дальнейшим действиям](#next-steps). Далее показано, как установить подключение и выполнить запрос к базе данных с помощью разных методов.

Чтобы удалить группу ресурсов, сделайте следующее:

### <a name="azure-portal"></a>Портал Azure

1. На [портале Azure](https://portal.azure.com) найдите и выберите элемент **Группы ресурсов**.
1. В списке групп ресурсов выберите имя нужной группы ресурсов.
1. На странице **Обзор** для этой группы ресурсов выберите **Удалить группу ресурсов**.
1. В диалоговом окне подтверждения введите имя группы ресурсов и щелкните **Удалить**.

### <a name="powershell"></a>PowerShell

```azurepowershell-interactive
$resourceGroupName = Read-Host -Prompt "Enter the Resource Group name"
Remove-AzResourceGroup -Name $resourceGroupName
Write-Host "Press [ENTER] to continue..."
```

### <a name="cli"></a>CLI

```azurecli-interactive
echo "Enter the Resource Group name:" &&
read resourceGroupName &&
az group delete --name $resourceGroupName &&
echo "Press [ENTER] to continue ..."
```
---

## <a name="next-steps"></a>Дальнейшие действия

Пошаговые инструкции по созданию шаблона ARM см. в следующей статье:

> [!div class="nextstepaction"]
> [Создание и развертывание первого шаблона ARM](../../azure-resource-manager/templates/template-tutorial-create-first-template.md)

Пошаговое руководство по созданию приложения в Службе приложений с помощью MySQL представлено в статье:

> [!div class="nextstepaction"]
>[Создание веб-приложения PHP (Laravel) с использованием MySQL](tutorial-php-database-app.md)
