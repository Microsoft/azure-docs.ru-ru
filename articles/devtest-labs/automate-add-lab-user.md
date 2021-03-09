---
title: Автоматизация добавления пользователя лаборатории в Azure DevTest Labs | Документация Майкрософт
description: В этой статье показано, как автоматизировать добавление пользователя в лабораторию в Azure DevTest Labs с помощью шаблонов Azure Resource Manager, PowerShell и интерфейса командной строки.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: 6dddf06289da79e16cbd7e64869fa77f0a40dd22
ms.sourcegitcommit: 15d27661c1c03bf84d3974a675c7bd11a0e086e6
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/09/2021
ms.locfileid: "102508832"
---
# <a name="automate-adding-a-lab-user-to-a-lab-in-azure-devtest-labs"></a>Автоматизация добавления пользователя лаборатории в лабораторию в Azure DevTest Labs
Azure DevTest Labs позволяет быстро создавать среды самообслуживания для разработки и тестирования с помощью портал Azure. Однако если у вас есть несколько команд и несколько экземпляров DevTest Labs, Автоматизация процесса создания позволит сэкономить время. [Шаблоны Azure Resource Manager](https://github.com/Azure/azure-devtestlab/tree/master/Environments) позволяют создавать лабораторные и лабораторные виртуальные машины, пользовательские образы, формулы и добавлять пользователей в автоматическом режиме. В этой статье особое внимание уделяется добавлению пользователей в экземпляр DevTest Labs.

Чтобы добавить пользователя в лабораторию, добавьте пользователя в роль **пользователя DevTest Labs** для лаборатории. В этой статье показано, как автоматизировать добавление пользователя в лабораторию с помощью одного из следующих способов.

- Шаблоны Azure Resource Manager
- Командлеты Azure PowerShell 
- Azure CLI.

## <a name="use-azure-resource-manager-templates"></a>Использование шаблонов Azure Resource Manager
В следующем примере шаблона диспетчер ресурсов указывается пользователь, добавляемый в роль **пользователя DevTest Labs** лаборатории. 

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "principalId": {
      "type": "string",
      "metadata": {
        "description": "The objectId of the user, group, or service principal for the role."
      }
    },
    "labName": {
      "type": "string",
      "metadata": {
        "description": "The name of the lab instance to be created."
      }
    },
    "roleAssignmentGuid": {
      "type": "string",
      "metadata": {
        "description": "Guid to use as the name for the role assignment."
      }
    }
  },
  "variables": {
    "devTestLabUserRoleId": "[concat('/subscriptions/', subscription().subscriptionId, '/providers/Microsoft.Authorization/roleDefinitions/111111111-0000-0000-11111111111111111')]",
    "fullDevTestLabUserRoleName": "[concat(parameters('labName'), '/Microsoft.Authorization/', parameters('roleAssignmentGuid'))]",
    "roleScope": "[concat('/subscriptions/', subscription().subscriptionId, '/resourceGroups/', resourceGroup().name, '/providers/Microsoft.DevTestLab/labs/', parameters('labName'))]"
  },
  "resources": [
    {
      "apiVersion": "2016-05-15",
      "type": "Microsoft.DevTestLab/labs",
      "name": "[parameters('labName')]",
      "location": "[resourceGroup().location]"
    },
    {
      "apiVersion": "2016-07-01",
      "type": "Microsoft.DevTestLab/labs/providers/roleAssignments",
      "name": "[variables('fullDevTestLabUserRoleName')]",
      "properties": {
        "roleDefinitionId": "[variables('devTestLabUserRoleId')]",
        "principalId": "[parameters('principalId')]",
        "scope": "[variables('roleScope')]"
      },
      "dependsOn": [
        "[resourceId('Microsoft.DevTestLab/labs', parameters('labName'))]"
      ]
    }
  ]
}

```

При назначении роли в том же шаблоне, который создает лабораторию, не забудьте добавить зависимость между ресурсом назначения роли и лабораторией. Дополнительные сведения см. [в статье Определение зависимостей в Azure Resource Manager шаблонов](../azure-resource-manager/templates/define-resource-dependency.md) .

### <a name="role-assignment-resource-information"></a>Сведения о ресурсах назначения ролей
Ресурс назначения роли должен указывать тип и имя.

Прежде всего следует отметить, что тип ресурса не `Microsoft.Authorization/roleAssignments` так, как для группы ресурсов.  Вместо этого тип ресурса соответствует шаблону `{provider-namespace}/{resource-type}/providers/roleAssignments` . В этом случае типом ресурса будет `Microsoft.DevTestLab/labs/providers/roleAssignments` .

Имя назначения роли должно быть глобально уникальным.  Имя назначения использует шаблон `{labName}/Microsoft.Authorization/{newGuid}` . `newGuid`— Это значение параметра для шаблона. Это гарантирует уникальность имени назначения роли. Так как для создания идентификаторов GUID не существует функций шаблона, необходимо создать GUID самостоятельно с помощью любого средства генератора GUID.  

В шаблоне имя для назначения роли определяется `fullDevTestLabUserRoleName` переменной. Точная строка из шаблона:

```json
"fullDevTestLabUserRoleName": "[concat(parameters('labName'), '/Microsoft.Authorization/', parameters('roleAssignmentGuid'))]"
```


### <a name="role-assignment-resource-properties"></a>Свойства ресурса назначения роли
Назначение роли определяет три свойства. Для этого требуется `roleDefinitionId` , `principalId` и `scope` .

### <a name="role-definition"></a>Определение роли
Идентификатор определения роли — это идентификатор строки для существующего определения роли. Идентификатор роли имеет вид `/subscriptions/{subscription-id}/providers/Microsoft.Authorization/roleDefinitions/{role-definition-id}` . 

Идентификатор подписки получается с помощью `subscription().subscriptionId` функции-шаблона.  

Необходимо получить определение роли для `DevTest Labs User` встроенной роли. Чтобы получить идентификатор GUID для роли [пользователя DevTest Labs](../role-based-access-control/built-in-roles.md#devtest-labs-user) , можно использовать [назначения ролей REST API](/rest/api/authorization/roleassignments) или командлет [Get-азроледефинитион](/powershell/module/az.resources/get-azroledefinition) .

```powershell
$dtlUserRoleDefId = (Get-AzRoleDefinition -Name "DevTest Labs User").Id
```

Идентификатор роли определяется в разделе Variables и с именем `devTestLabUserRoleId` . В шаблоне для идентификатора роли задано значение: 111111111-0000-0000-11111111111111111. 

```json
"devTestLabUserRoleId": "[concat('/subscriptions/', subscription().subscriptionId, '/providers/Microsoft.Authorization/roleDefinitions/111111111-0000-0000-11111111111111111')]",
```

### <a name="principal-id"></a>Идентификатор субъекта
ИДЕНТИФИКАТОР субъекта — это идентификатор объекта Active Directory пользователя, группы или субъекта-службы, который вы хотите добавить в лабораторию в качестве пользователя лаборатории. Шаблон использует в `ObjectId` качестве параметра.

Идентификатор ObjectId можно получить с помощью командлетов PowerShell [Get-AzureRMADUser](/powershell/module/azurerm.resources/get-azurermaduser?view=azurermps-6.13.0), [Get-Азурермадграуп или [Get-AzureRMADServicePrincipal](/powershell/module/azurerm.resources/get-azurermadserviceprincipal?view=azurermps-6.13.0) . Эти командлеты возвращают один или список объектов Active Directory, имеющих свойство ID, которое является нужным ИДЕНТИФИКАТОРом объекта. В следующем примере показано, как получить идентификатор объекта одного пользователя в компании.

```powershell
$userObjectId = (Get-AzureRmADUser -UserPrincipalName ‘email@company.com').Id
```

Можно также использовать командлеты PowerShell Azure Active Directory, включающие [Get-MsolUser](/powershell/module/msonline/get-msoluser?view=azureadps-1.0), [Get-MsolGroup](/powershell/module/msonline/get-msolgroup?view=azureadps-1.0)и [Get-MsolServicePrincipal](/powershell/module/msonline/get-msolserviceprincipal?view=azureadps-1.0).

### <a name="scope"></a>Область
Область указывает ресурс или группу ресурсов, к которым должно применяться назначение ролей. Для ресурсов область имеет вид: `/subscriptions/{subscription-id}/resourceGroups/{resource-group-name}/providers/{provider-namespace}/{resource-type}/{resource-name}` . Шаблон использует `subscription().subscriptionId` функцию для заполнения `subscription-id` части и `resourceGroup().name` функции шаблона для заполнения `resource-group-name` части. Использование этих функций означает, что лаборатория, которой назначается роль, должна существовать в текущей подписке и той же группе ресурсов, в которую выполняется развертывание шаблона. Последняя часть `resource-name` — это имя лаборатории. Это значение получается через параметр шаблона в этом примере. 

Область роли в шаблоне: 

```json
"roleScope": "[concat('/subscriptions/', subscription().subscriptionId, '/resourceGroups/', resourceGroup().name, '/providers/Microsoft.DevTestLab/labs/', parameters('labName'))]"
```

### <a name="deploying-the-template"></a>Развертывание шаблона
Сначала создайте файл параметров (например, azuredeploy.parameters.json), который будет передавать значения для параметров в шаблоне диспетчер ресурсов. 

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "principalId": {
      "value": "11111111-1111-1111-1111-111111111111"
    },
    "labName": {
      "value": "MyLab"
    },
    "roleAssignmentGuid": {
      "value": "22222222-2222-2222-2222-222222222222"
    }
  }
}
```

Затем используйте командлет PowerShell [New-AzureRmResourceGroupDeployment](/powershell/module/azurerm.resources/new-azurermresourcegroupdeployment?view=azurermps-6.13.0) , чтобы развернуть шаблон диспетчер ресурсов. В следующем примере команда назначает пользователю, группе или субъекту-службе роль пользователя DevTest Labs для лаборатории.

```powershell
New-AzureRmResourceGroupDeployment -Name "MyLabResourceGroup-$(New-Guid)" -ResourceGroupName 'MyLabResourceGroup' -TemplateParameterFile .\azuredeploy.parameters.json -TemplateFile .\azuredeploy.json
```

Важно отметить, что имя развертывания группы и идентификатор GUID назначения роли должны быть уникальными. При попытке развернуть назначение ресурса с идентификатором GUID, не являющимся уникальным, вы получите `RoleAssignmentUpdateNotPermitted` сообщение об ошибке.

Если вы планируете использовать шаблон несколько раз, чтобы добавить несколько Active Directory объектов к роли пользователя DevTest Labs для вашей лаборатории, рассмотрите возможность использования динамических объектов в команде PowerShell. В следующем примере используется командлет [New-GUID](/powershell/module/Microsoft.PowerShell.Utility/New-Guid?view=powershell-5.0) для динамического указания имени развертывания группы ресурсов и GUID назначения роли.

```powershell
New-AzureRmResourceGroupDeployment -Name "MyLabResourceGroup-$(New-Guid)" -ResourceGroupName 'MyLabResourceGroup' -TemplateFile .\azuredeploy.json -roleAssignmentGuid "$(New-Guid)" -labName "MyLab" -principalId "11111111-1111-1111-1111-111111111111"
```

## <a name="use-azure-powershell"></a>Использование Azure PowerShell
Как обсуждалось в статье Введение, вы создаете новое назначение роли Azure, чтобы добавить пользователя в роль **пользователя DevTest Labs** для лаборатории. В PowerShell это можно сделать с помощью командлета [New-AzureRMRoleAssignment](/powershell/module/azurerm.resources/new-azurermroleassignment?view=azurermps-6.13.0) . Этот командлет имеет множество необязательных параметров для обеспечения гибкости. `ObjectId`, `SigninName` Или `ServicePrincipalName` можно указать как объект, которому предоставляются разрешения.  

Ниже приведен пример команды Azure PowerShell, которая добавляет пользователя к роли пользователя DevTest Labs в указанной лаборатории.

```powershell
New-AzureRmRoleAssignment -UserPrincipalName <email@company.com> -RoleDefinitionName 'DevTest Labs User' -ResourceName '<Lab Name>' -ResourceGroupName '<Resource Group Name>' -ResourceType 'Microsoft.DevTestLab/labs'
```

Чтобы указать ресурс, которому предоставляются разрешения, можно задать сочетанием `ResourceName` `ResourceType` `ResourceGroup` параметров, или `scope` . При использовании любого сочетания параметров предоставьте командлету достаточно сведений для уникальной идентификации объекта Active Directory (пользователя, группы или субъекта-службы), области (группы ресурсов или ресурса) и определения роли.

## <a name="use-azure-command-line-interface-cli"></a>Использование интерфейса командной строки Azure (CLI)
В Azure CLI Добавление пользователя лаборатории в лабораторию выполняется с помощью `az role assignment create` команды. Дополнительные сведения о командлетах Azure CLI см. в статье [Добавление и удаление назначений ролей Azure с помощью Azure CLI](../role-based-access-control/role-assignments-cli.md).

Объект, которому предоставляется доступ, может быть задан `objectId` `signInName` параметрами,, `spn` . Лаборатория, к которой предоставляется доступ к объекту, может быть идентифицирована по `scope` URL-адресу или сочетанием `resource-name` `resource-type` параметров, и `resource-group` .

В следующем примере Azure CLI показано, как добавить пользователя в роль пользователя DevTest Labs для указанной лаборатории.  

```azurecli
az role assignment create --roleName "DevTest Labs User" --signInName <email@company.com> -–resource-name "<Lab Name>" --resource-type “Microsoft.DevTestLab/labs" --resource-group "<Resource Group Name>"
```

## <a name="next-steps"></a>Дальнейшие действия
См. следующие статьи:

- [Создание виртуальных машин и управление ими с DevTest Labs с использованием Azure CLI](devtest-lab-vmcli.md)
- [Создание виртуальной машины с DevTest Labs с помощью Azure PowerShell](devtest-lab-vm-powershell.md)
- [Использование программ командной строки для запуска и завершения Azure DevTest Labs виртуальных машин](use-command-line-start-stop-virtual-machines.md)

