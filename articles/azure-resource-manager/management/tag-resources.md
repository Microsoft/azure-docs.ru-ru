---
title: Разметка ресурсов, групп ресурсов и подписок для логической организации
description: Здесь описано, как применить теги, чтобы организовать ресурсы Azure для выставления счетов и управления.
ms.topic: conceptual
ms.date: 01/04/2021
ms.custom: devx-track-azurecli
ms.openlocfilehash: fb93673b643fd13efe9ffea148c5fb1d072f9e05
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98896229"
---
# <a name="use-tags-to-organize-your-azure-resources-and-management-hierarchy"></a>Использование тегов для Организации ресурсов Azure и иерархии управления

Вы примените теги к ресурсам, группам ресурсов и подпискам Azure, чтобы логически упорядочить их в таксономии. Каждый тег состоит из пары "имя — значение". Например, имя Environment и значение Production можно применить ко всем ресурсам в рабочей среде.

Рекомендации по реализации стратегии тегов см. в разделе [руководство по именованию ресурсов и созданию тегов](/azure/cloud-adoption-framework/decision-guides/resource-tagging/?toc=/azure/azure-resource-manager/management/toc.json).

> [!IMPORTANT]
> В именах тегов не учитывается регистр для операций. Обновляется или извлекается тег с именем тега, независимо от регистра. Однако поставщик ресурсов может оставить регистр, который вы задаюте для имени тега. Вы увидите этот регистр в отчетах о затратах.
> 
> В значениях тегов учитывается регистр.

[!INCLUDE [Handle personal data](../../../includes/gdpr-intro-sentence.md)]

## <a name="required-access"></a>Требуемый доступ

Есть два способа получить необходимый доступ к ресурсам тегов.

- Вы можете иметь доступ на запись к типу ресурса **Microsoft. Resources/Tags** . Этот доступ позволяет пометить любой ресурс, даже если у вас нет доступа к самому ресурсу. Роль [участника тега](../../role-based-access-control/built-in-roles.md#tag-contributor) предоставляет этот доступ. В настоящее время роль участника тега не может применять теги к ресурсам или группам ресурсов на портале. Он может применять теги к подпискам на портале. Она поддерживает все операции с тегами с помощью PowerShell и REST API.  

- Вы можете иметь доступ на запись к самому ресурсу. Роль [участника](../../role-based-access-control/built-in-roles.md#contributor) предоставляет необходимый доступ для применения тегов к любой сущности. Чтобы применить теги только к одному типу ресурсов, используйте роль участника для этого ресурса. Например, чтобы применить теги к виртуальным машинам, используйте [Участник виртуальных машин](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor).

## <a name="powershell"></a>PowerShell

### <a name="apply-tags"></a>Применить Теги

Azure PowerShell предлагает две команды для применения тегов- [New-азтаг](/powershell/module/az.resources/new-aztag) и [Update-азтаг](/powershell/module/az.resources/update-aztag). Необходимо иметь модуль AZ. Resources 1.12.0 или более поздней версии. Версию можно проверить с помощью `Get-Module Az.Resources` . Вы можете установить этот модуль или [установить Azure PowerShell](/powershell/azure/install-az-ps) 3.6.1 или более поздней версии.

**New-азтаг** заменяет все теги в ресурсе, группе ресурсов или подписке. При вызове команды передайте идентификатор ресурса сущности, которую вы хотите пометить.

В следующем примере набор тегов применяется к учетной записи хранения.

```azurepowershell-interactive
$tags = @{"Dept"="Finance"; "Status"="Normal"}
$resource = Get-AzResource -Name demoStorage -ResourceGroup demoGroup
New-AzTag -ResourceId $resource.id -Tag $tags
```

После выполнения команды Обратите внимание, что ресурс имеет два тега.

```output
Properties :
        Name    Value
        ======  =======
        Dept    Finance
        Status  Normal
```

Если выполнить команду еще раз, но на этот раз с другими тегами, обратите внимание, что предыдущие Теги удалены.

```azurepowershell-interactive
$tags = @{"Team"="Compliance"; "Environment"="Production"}
New-AzTag -ResourceId $resource.id -Tag $tags
```

```output
Properties :
        Name         Value
        ===========  ==========
        Environment  Production
        Team         Compliance
```

Чтобы добавить теги к ресурсу, который уже содержит теги, используйте **Update-азтаг**. Задайте для параметра **-Operation** значение **Merge**.

```azurepowershell-interactive
$tags = @{"Dept"="Finance"; "Status"="Normal"}
Update-AzTag -ResourceId $resource.id -Tag $tags -Operation Merge
```

Обратите внимание, что два новых тега были добавлены к двум существующим тегам.

```output
Properties :
        Name         Value
        ===========  ==========
        Status       Normal
        Dept         Finance
        Team         Compliance
        Environment  Production
```

Каждое имя тега может иметь только одно значение. Если указать новое значение для тега, старое значение будет заменено даже при использовании операции MERGE. В следующем примере тег Status изменяется с нормального на зеленый.

```azurepowershell-interactive
$tags = @{"Status"="Green"}
Update-AzTag -ResourceId $resource.id -Tag $tags -Operation Merge
```

```output
Properties :
        Name         Value
        ===========  ==========
        Status       Green
        Dept         Finance
        Team         Compliance
        Environment  Production
```

Если для параметра **-Operation** задано значение **Replace**, существующие теги заменяются новым набором тегов.

```azurepowershell-interactive
$tags = @{"Project"="ECommerce"; "CostCenter"="00123"; "Team"="Web"}
Update-AzTag -ResourceId $resource.id -Tag $tags -Operation Replace
```

В ресурсе остаются только новые теги.

```output
Properties :
        Name        Value
        ==========  =========
        CostCenter  00123
        Team        Web
        Project     ECommerce
```

Одни и те же команды также работают с группами ресурсов или подписками. Вы передаете идентификатор для группы ресурсов или подписки, которые необходимо пометить.

Чтобы добавить новый набор тегов в группу ресурсов, используйте:

```azurepowershell-interactive
$tags = @{"Dept"="Finance"; "Status"="Normal"}
$resourceGroup = Get-AzResourceGroup -Name demoGroup
New-AzTag -ResourceId $resourceGroup.ResourceId -tag $tags
```

Чтобы обновить теги для группы ресурсов, используйте:

```azurepowershell-interactive
$tags = @{"CostCenter"="00123"; "Environment"="Production"}
$resourceGroup = Get-AzResourceGroup -Name demoGroup
Update-AzTag -ResourceId $resourceGroup.ResourceId -Tag $tags -Operation Merge
```

Чтобы добавить новый набор тегов к подписке, используйте:

```azurepowershell-interactive
$tags = @{"CostCenter"="00123"; "Environment"="Dev"}
$subscription = (Get-AzSubscription -SubscriptionName "Example Subscription").Id
New-AzTag -ResourceId "/subscriptions/$subscription" -Tag $tags
```

Чтобы обновить теги для подписки, используйте:

```azurepowershell-interactive
$tags = @{"Team"="Web Apps"}
$subscription = (Get-AzSubscription -SubscriptionName "Example Subscription").Id
Update-AzTag -ResourceId "/subscriptions/$subscription" -Tag $tags -Operation Merge
```

В группе ресурсов может быть несколько ресурсов с одним и тем же именем. В этом случае можно задать каждый ресурс с помощью следующих команд:

```azurepowershell-interactive
$resource = Get-AzResource -ResourceName sqlDatabase1 -ResourceGroupName examplegroup
$resource | ForEach-Object { Update-AzTag -Tag @{ "Dept"="IT"; "Environment"="Test" } -ResourceId $_.ResourceId -Operation Merge }
```

### <a name="list-tags"></a>Вывод списка тегов

Чтобы получить теги для ресурса, группы ресурсов или подписки, используйте команду [Get-азтаг](/powershell/module/az.resources/get-aztag) и передайте идентификатор ресурса для сущности.

Чтобы просмотреть теги для ресурса, используйте:

```azurepowershell-interactive
$resource = Get-AzResource -Name demoStorage -ResourceGroup demoGroup
Get-AzTag -ResourceId $resource.id
```

Чтобы просмотреть теги для группы ресурсов, используйте:

```azurepowershell-interactive
$resourceGroup = Get-AzResourceGroup -Name demoGroup
Get-AzTag -ResourceId $resourceGroup.ResourceId
```

Чтобы просмотреть теги для подписки, используйте:

```azurepowershell-interactive
$subscription = (Get-AzSubscription -SubscriptionName "Example Subscription").Id
Get-AzTag -ResourceId "/subscriptions/$subscription"
```

### <a name="list-by-tag"></a>Список по тегу

Чтобы получить ресурсы с указанными именем и значением тега, используйте:

```azurepowershell-interactive
(Get-AzResource -Tag @{ "CostCenter"="00123"}).Name
```

Чтобы получить ресурсы с заданным именем тега и любым значением тега, используйте:

```azurepowershell-interactive
(Get-AzResource -TagName "Dept").Name
```

Чтобы получить группы ресурсов с указанными именем и значением тега, используйте:

```azurepowershell-interactive
(Get-AzResourceGroup -Tag @{ "CostCenter"="00123" }).ResourceGroupName
```

### <a name="remove-tags"></a>Удалить теги

Чтобы удалить определенные теги, используйте **Update-азтаг** и Set **-Operation** для **удаления**. Передайте теги, которые нужно удалить.

```azurepowershell-interactive
$removeTags = @{"Project"="ECommerce"; "Team"="Web"}
Update-AzTag -ResourceId $resource.id -Tag $removeTags -Operation Delete
```

Указанные теги удаляются.

```output
Properties :
        Name        Value
        ==========  =====
        CostCenter  00123
```

Чтобы удалить все теги, используйте команду [Remove-азтаг](/powershell/module/az.resources/remove-aztag) .

```azurepowershell-interactive
$subscription = (Get-AzSubscription -SubscriptionName "Example Subscription").Id
Remove-AzTag -ResourceId "/subscriptions/$subscription"
```

## <a name="azure-cli"></a>Azure CLI

### <a name="apply-tags"></a>Применить Теги

Azure CLI предлагает две команды для применения тегов — [AZ Tag Create](/cli/azure/tag#az_tag_create) и [AZ Tag Update](/cli/azure/tag#az_tag_update). Необходимо иметь Azure CLI 2.10.0 или более поздней версии. Версию можно проверить с помощью `az version` . Сведения об обновлении или установке см. [в разделе установка Azure CLI](/cli/azure/install-azure-cli).

**Тег AZ Create** заменяет все теги в ресурсе, группе ресурсов или подписке. При вызове команды передайте идентификатор ресурса сущности, которую вы хотите пометить.

В следующем примере набор тегов применяется к учетной записи хранения.

```azurecli-interactive
resource=$(az resource show -g demoGroup -n demoStorage --resource-type Microsoft.Storage/storageAccounts --query "id" --output tsv)
az tag create --resource-id $resource --tags Dept=Finance Status=Normal
```

После выполнения команды Обратите внимание, что ресурс имеет два тега.

```output
"properties": {
  "tags": {
    "Dept": "Finance",
    "Status": "Normal"
  }
},
```

Если выполнить команду еще раз, но на этот раз с другими тегами, обратите внимание, что предыдущие Теги удалены.

```azurecli-interactive
az tag create --resource-id $resource --tags Team=Compliance Environment=Production
```

```output
"properties": {
  "tags": {
    "Environment": "Production",
    "Team": "Compliance"
  }
},
```

Чтобы добавить теги к ресурсу, который уже содержит теги, используйте `az tag update` . Установите для параметра `--operation` значение `Merge`.

```azurecli-interactive
az tag update --resource-id $resource --operation Merge --tags Dept=Finance Status=Normal
```

Обратите внимание, что два новых тега были добавлены к двум существующим тегам.

```output
"properties": {
  "tags": {
    "Dept": "Finance",
    "Environment": "Production",
    "Status": "Normal",
    "Team": "Compliance"
  }
},
```

Каждое имя тега может иметь только одно значение. Если указать новое значение для тега, старое значение будет заменено даже при использовании операции MERGE. В следующем примере тег Status изменяется с нормального на зеленый.

```azurecli-interactive
az tag update --resource-id $resource --operation Merge --tags Status=Green
```

```output
"properties": {
  "tags": {
    "Dept": "Finance",
    "Environment": "Production",
    "Status": "Green",
    "Team": "Compliance"
  }
},
```

Если `--operation` для параметра задано значение `Replace` , существующие теги заменяются новым набором тегов.

```azurecli-interactive
az tag update --resource-id $resource --operation Replace --tags Project=ECommerce CostCenter=00123 Team=Web
```

В ресурсе остаются только новые теги.

```output
"properties": {
  "tags": {
    "CostCenter": "00123",
    "Project": "ECommerce",
    "Team": "Web"
  }
},
```

Одни и те же команды также работают с группами ресурсов или подписками. Вы передаете идентификатор для группы ресурсов или подписки, которые необходимо пометить.

Чтобы добавить новый набор тегов в группу ресурсов, используйте:

```azurecli-interactive
group=$(az group show -n demoGroup --query id --output tsv)
az tag create --resource-id $group --tags Dept=Finance Status=Normal
```

Чтобы обновить теги для группы ресурсов, используйте:

```azurecli-interactive
az tag update --resource-id $group --operation Merge --tags CostCenter=00123 Environment=Production
```

Чтобы добавить новый набор тегов к подписке, используйте:

```azurecli-interactive
sub=$(az account show --subscription "Demo Subscription" --query id --output tsv)
az tag create --resource-id /subscriptions/$sub --tags CostCenter=00123 Environment=Dev
```

Чтобы обновить теги для подписки, используйте:

```azurecli-interactive
az tag update --resource-id /subscriptions/$sub --operation Merge --tags Team="Web Apps"
```

### <a name="list-tags"></a>Вывод списка тегов

Чтобы получить теги для ресурса, группы ресурсов или подписки, используйте команду [AZ Tag List](/cli/azure/tag#az_tag_list) и передайте идентификатор ресурса для сущности.

Чтобы просмотреть теги для ресурса, используйте:

```azurecli-interactive
resource=$(az resource show -g demoGroup -n demoStorage --resource-type Microsoft.Storage/storageAccounts --query "id" --output tsv)
az tag list --resource-id $resource
```

Чтобы просмотреть теги для группы ресурсов, используйте:

```azurecli-interactive
group=$(az group show -n demoGroup --query id --output tsv)
az tag list --resource-id $group
```

Чтобы просмотреть теги для подписки, используйте:

```azurecli-interactive
sub=$(az account show --subscription "Demo Subscription" --query id --output tsv)
az tag list --resource-id /subscriptions/$sub
```

### <a name="list-by-tag"></a>Список по тегу

Чтобы получить ресурсы с указанными именем и значением тега, используйте:

```azurecli-interactive
az resource list --tag CostCenter=00123 --query [].name
```

Чтобы получить ресурсы с заданным именем тега и любым значением тега, используйте:

```azurecli-interactive
az resource list --tag Team --query [].name
```

Чтобы получить группы ресурсов с указанными именем и значением тега, используйте:

```azurecli-interactive
az group list --tag Dept=Finance
```

### <a name="remove-tags"></a>Удалить теги

Чтобы удалить определенные теги, используйте `az tag update` и задайте `--operation` для значение `Delete` . Передайте теги, которые нужно удалить.

```azurecli-interactive
az tag update --resource-id $resource --operation Delete --tags Project=ECommerce Team=Web
```

Указанные теги удаляются.

```output
"properties": {
  "tags": {
    "CostCenter": "00123"
  }
},
```

Чтобы удалить все теги, используйте команду [AZ Tag Delete](/cli/azure/tag#az_tag_delete) .

```azurecli-interactive
az tag delete --resource-id $resource
```

### <a name="handling-spaces"></a>Обработка пробелов

Если имена или значения тегов содержат пробелы, заключите их в двойные кавычки.

```azurecli-interactive
az tag update --resource-id $group --operation Merge --tags "Cost Center"=Finance-1222 Location="West US"
```

## <a name="arm-templates"></a>Шаблоны ARM

Вы можете помечать ресурсы, группы ресурсов и подписки во время развертывания с помощью шаблона Azure Resource Manager (шаблон ARM).

> [!NOTE]
> Теги, применяемые с помощью шаблона ARM, перезапишут все существующие теги.

### <a name="apply-values"></a>Применить значения

В следующем примере выполняется развертывание учетной записи хранения с тремя тегами. Два тега ( `Dept` и `Environment` ) задаются как литеральные значения. Для одного тега ( `LastDeployed` ) задан параметр, который по умолчанию имеет текущую дату.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "utcShort": {
            "type": "string",
            "defaultValue": "[utcNow('d')]"
        },
        "location": {
            "type": "string",
            "defaultValue": "[resourceGroup().location]"
        }
    },
    "resources": [
        {
            "apiVersion": "2019-04-01",
            "type": "Microsoft.Storage/storageAccounts",
            "name": "[concat('storage', uniqueString(resourceGroup().id))]",
            "location": "[parameters('location')]",
            "tags": {
                "Dept": "Finance",
                "Environment": "Production",
                "LastDeployed": "[parameters('utcShort')]"
            },
            "sku": {
                "name": "Standard_LRS"
            },
            "kind": "Storage",
            "properties": {}
        }
    ]
}
```

### <a name="apply-an-object"></a>Применение объекта

Можно определить параметр объекта, который хранит несколько тегов, и применить этот объект к элементу тега. Такой подход обеспечивает большую гибкость, чем предыдущий пример, поскольку объект может иметь различные свойства. Каждое свойство в объекте становится отдельным тегом ресурса. В следующем примере содержится параметр с именем `tagValues`, который применяется к элементу тега.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "location": {
            "type": "string",
            "defaultValue": "[resourceGroup().location]"
        },
        "tagValues": {
            "type": "object",
            "defaultValue": {
                "Dept": "Finance",
                "Environment": "Production"
            }
        }
    },
    "resources": [
        {
            "apiVersion": "2019-04-01",
            "type": "Microsoft.Storage/storageAccounts",
            "name": "[concat('storage', uniqueString(resourceGroup().id))]",
            "location": "[parameters('location')]",
            "tags": "[parameters('tagValues')]",
            "sku": {
                "name": "Standard_LRS"
            },
            "kind": "Storage",
            "properties": {}
        }
    ]
}
```

### <a name="apply-a-json-string"></a>Применение строки JSON

Для хранения большого количества значений в одном теге примените строку JSON, представляющую значения. Вся строка JSON хранится в виде одного тега, длина которого не должна превышать 256 символов. В следующем примере приведен один тег с именем `CostCenter`, содержащий несколько значений из строки JSON.  

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "location": {
            "type": "string",
            "defaultValue": "[resourceGroup().location]"
        }
    },
    "resources": [
        {
            "apiVersion": "2019-04-01",
            "type": "Microsoft.Storage/storageAccounts",
            "name": "[concat('storage', uniqueString(resourceGroup().id))]",
            "location": "[parameters('location')]",
            "tags": {
                "CostCenter": "{\"Dept\":\"Finance\",\"Environment\":\"Production\"}"
            },
            "sku": {
                "name": "Standard_LRS"
            },
            "kind": "Storage",
            "properties": {}
        }
    ]
}
```

### <a name="apply-tags-from-resource-group"></a>Применение тегов из группы ресурсов

Чтобы применить теги из группы ресурсов к ресурсу, используйте функцию [resourceGroup ()](../templates/template-functions-resource.md#resourcegroup) . При получении значения тега используйте `tags[tag-name]` синтаксис вместо `tags.tag-name` синтаксиса, так как некоторые символы не анализируются правильно в точечной нотации.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "location": {
            "type": "string",
            "defaultValue": "[resourceGroup().location]"
        }
    },
    "resources": [
        {
            "apiVersion": "2019-04-01",
            "type": "Microsoft.Storage/storageAccounts",
            "name": "[concat('storage', uniqueString(resourceGroup().id))]",
            "location": "[parameters('location')]",
            "tags": {
                "Dept": "[resourceGroup().tags['Dept']]",
                "Environment": "[resourceGroup().tags['Environment']]"
            },
            "sku": {
                "name": "Standard_LRS"
            },
            "kind": "Storage",
            "properties": {}
        }
    ]
}
```

### <a name="apply-tags-to-resource-groups-or-subscriptions"></a>Применение тегов к группам ресурсов или подпискам

Вы можете добавить теги в группу ресурсов или подписку, развернув тип ресурса **Microsoft. Resources/Tags** . Теги применяются к целевой группе ресурсов или подписке для развертывания. Каждый раз при развертывании шаблона заменяются все теги, которые были ранее применены.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "tagName": {
            "type": "string",
            "defaultValue": "TeamName"
        },
        "tagValue": {
            "type": "string",
            "defaultValue": "AppTeam1"
        }
    },
    "variables": {},
    "resources": [
        {
            "type": "Microsoft.Resources/tags",
            "name": "default",
            "apiVersion": "2019-10-01",
            "dependsOn": [],
            "properties": {
                "tags": {
                    "[parameters('tagName')]": "[parameters('tagValue')]"
                }
            }
        }
    ]
}
```

Чтобы применить теги к группе ресурсов, используйте либо PowerShell, либо Azure CLI. Разверните в группе ресурсов, которую необходимо пометить.

```azurepowershell-interactive
New-AzResourceGroupDeployment -ResourceGroupName exampleGroup -TemplateFile https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/tags.json
```

```azurecli-interactive
az deployment group create --resource-group exampleGroup --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/tags.json
```

Чтобы применить теги к подписке, используйте либо PowerShell, либо Azure CLI. Выполните развертывание в подписке, которую нужно пометить.

```azurepowershell-interactive
New-AzSubscriptionDeployment -name tagresourcegroup -Location westus2 -TemplateUri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/tags.json
```

```azurecli-interactive
az deployment sub create --name tagresourcegroup --location westus2 --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/tags.json
```

Дополнительные сведения о развертывании подписок см. в статье [Создание групп ресурсов и ресурсов на уровне подписки](../templates/deploy-to-subscription.md).

Следующий шаблон добавляет теги из объекта в группу ресурсов или подписку.

```json
"$schema": "https://schema.management.azure.com/schemas/2018-05-01/subscriptionDeploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "tags": {
            "type": "object",
            "defaultValue": {
                "TeamName": "AppTeam1",
                "Dept": "Finance",
                "Environment": "Production"
            }
        }
    },
    "variables": {},
    "resources": [
        {
            "type": "Microsoft.Resources/tags",
            "name": "default",
            "apiVersion": "2019-10-01",
            "dependsOn": [],
            "properties": {
                "tags": "[parameters('tags')]"
            }
        }
    ]
}
```

## <a name="portal"></a>Портал

[!INCLUDE [resource-manager-tag-resource](../../../includes/resource-manager-tag-resources.md)]

## <a name="rest-api"></a>REST API

Для работы с тегами с помощью REST API Azure используйте:

* [Теги — создание или обновление в области действия](/rest/api/resources/tags/createorupdateatscope) (операция размещения)
* [Теги — обновление в области](/rest/api/resources/tags/updateatscope) (операция исправления)
* [Теги — получение в области действия](/rest/api/resources/tags/getatscope) (операция Get)
* [Теги — удаление в области действия](/rest/api/resources/tags/deleteatscope) (операция удаления)

## <a name="inherit-tags"></a>Наследование тегов

Теги, применяемые к группе ресурсов или подписке, не наследуются ресурсами. Чтобы применить теги из подписки или группы ресурсов к ресурсам, см. раздел [политики Azure — Теги](tag-policies.md).

## <a name="tags-and-billing"></a>Теги и выставление счетов

С помощью тегов можно группировать данные о выставлении счетов. Например, если у вас работает несколько виртуальных машин для разных организаций, то с помощью тегов можно группировать сведения об использовании по месту возникновения затрат. Кроме того, теги можно использовать для группирования затрат по среде выполнения (например, сведения о выставленных счетах за виртуальные машины, запущенные в рабочей среде).

Сведения о тегах можно получить, загрузив файл использования, файл данных с разделителями-запятыми (CSV), доступный в портал Azure. Дополнительные сведения см. в статье [Скачивание или просмотр счета на оплату и данных о ежедневном использовании в Azure](../../cost-management-billing/manage/download-azure-invoice-daily-usage-date.md). При скачивании файла сведений об использовании из Центра управления учетной записью Azure выберите **Версия 2**. Для служб, поддерживающих теги выставления счетов, эти теги отображаются в столбце **Теги**.

Подробнее об операциях REST API см. в [справочнике по REST API для выставления счетов Azure](/rest/api/billing/).

## <a name="limitations"></a>Ограничения

Действительны следующие ограничения для тегов.

* Не все типы ресурсов поддерживают теги. Сведения о возможности применения тегов к типу ресурса см. в статье о [поддержке тегов ресурсами Azure](tag-support.md).
* Каждый ресурс, Группа ресурсов и подписка могут иметь не более 50 пар "имя-значение" для тегов. Если необходимо применить больше тегов, чем максимально допустимое число, используйте строку JSON для значения тега. Строка JSON может содержать много значений, применяемых к одному имени тега. Группа ресурсов или подписка может содержать множество ресурсов, каждая из которых содержит 50 пар "имя-значение" для тегов.
* Имя тега ограничено 512 символами, а значение тега — 256 символами. Для учетных записей хранения имя тега ограничено 128 символами, а значение тега — 256 символами.
* Теги нельзя применять к классическим ресурсам, например к облачным службам Microsoft Azure.
* Имена тегов не могут содержать следующие символы: `<`, `>`, `%`, `&`, `\`, `?`, `/`

   > [!NOTE]
   > В настоящее время зоны Azure DNS и службы диспетчера трафика также не разрешают использовать пробелы в теге.
   >
   > Передняя дверца Azure не поддерживает использование `#` в имени тега.
   >
   > Служба автоматизации Azure и Azure CDN поддерживают только 15 тегов для ресурсов.

## <a name="next-steps"></a>Дальнейшие действия

* Не все типы ресурсов поддерживают теги. Сведения о возможности применения тегов к типу ресурса см. в статье о [поддержке тегов ресурсами Azure](tag-support.md).
* Рекомендации по реализации стратегии тегов см. в разделе [руководство по именованию ресурсов и созданию тегов](/azure/cloud-adoption-framework/decision-guides/resource-tagging/?toc=/azure/azure-resource-manager/management/toc.json).
