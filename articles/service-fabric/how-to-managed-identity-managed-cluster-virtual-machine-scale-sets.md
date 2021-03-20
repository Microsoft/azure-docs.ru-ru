---
title: Добавление управляемого удостоверения в тип узла управляемого кластера Service Fabric (Предварительная версия)
description: В этой статье показано, как добавить управляемое удостоверение в тип узла управляемого кластера Service Fabric
ms.topic: how-to
ms.date: 11/24/2020
ms.custom: references_regions
ms.openlocfilehash: 3ff5d66160ddbb037469378634826fd9eeae0c54
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100651652"
---
# <a name="add-a-managed-identity-to-a-service-fabric-managed-cluster-node-type-preview"></a>Добавление управляемого удостоверения в тип узла управляемого кластера Service Fabric (Предварительная версия)

Каждый тип узла в управляемом кластере Service Fabric поддерживается масштабируемым набором виртуальных машин. Чтобы разрешить использование управляемых удостоверений с типом узла управляемого кластера, в `vmManagedIdentity` определения типа узла было добавлено свойство, содержащее список удостоверений, которые могут использоваться `userAssignedIdentities` . Функция отражает, как управляемые удостоверения могут использоваться в неуправляемых кластерах, например, с помощью управляемого удостоверения с [Azure Key Vault расширением масштабируемого набора виртуальных машин](../virtual-machines/extensions/key-vault-windows.md).


Пример Service Fabric развертывания управляемого кластера, использующего управляемое удостоверение для типа узла, см. в [этом шаблоне](https://github.com/Azure-Samples/service-fabric-cluster-templates/tree/master/SF-Managed-Standard-SKU-1-NT-MI). Список поддерживаемых регионов см. в разделе [часто задаваемые вопросы об управляемом кластере](./faq-managed-cluster.md#what-regions-are-supported-in-the-preview).

> [!NOTE]
> Для этой функции сейчас поддерживаются только удостоверения, назначенные пользователем.

## <a name="prerequisites"></a>Предварительные требования

Перед началом работы

* Если у вас еще нет подписки Azure, создайте [бесплатную](https://azure.microsoft.com/free/) учетную запись Azure, прежде чем начинать работу.
* Если вы планируете использовать PowerShell, [установите](/cli/azure/install-azure-cli) Azure CLI, чтобы выполнить справочные команды CLI.

## <a name="create-a-user-assigned-managed-identity"></a>Создание управляемого удостоверения, назначаемого пользователем 

Назначаемое пользователем управляемое удостоверение может быть определено в разделе ресурсов шаблона Azure Resource Manager (ARM) для создания при развертывании:

```JSON
{ 
    "type": "Microsoft.ManagedIdentity/userAssignedIdentities", 
    "name": "[parameters('userAssignedIdentityName')]", 
    "apiVersion": "2018-11-30", 
    "location": "[resourceGroup().location]"  
},
```

или создать с помощью PowerShell:

```powershell
az group create --name <resourceGroupName> --location <location>
az identity create --name <userAssignedIdentityName> --resource-group <resourceGroupName>
```

## <a name="add-a-role-assignment-with-service-fabric-resource-provider"></a>Добавление назначения роли с помощью поставщика ресурсов Service Fabric

Добавьте назначение роли для управляемого удостоверения с помощью приложения поставщика ресурсов Service Fabric. Это назначение позволяет поставщику ресурсов Service Fabric назначить удостоверение масштабируемому набору виртуальных машин управляемого кластера. 

При необходимости следует использовать следующие значения:

|Имя|Соответствующее значение поставщика ресурсов Service Fabric|
|----|-------------------------------------|
|Идентификатор приложения|74cb6831-0dbb-4be1-8206-fd4df301cdc2|
|Идентификатор объекта|fbc587f2-66f5-4459-a027-bcd908b9d278|


|Имя определения роли|Идентификатор определения роли|
|----|-------------------------------------|
|Оператор управляемого удостоверения|f1a07417-d97a-45cb-824c-7a7467783830
|



Это назначение ролей можно определить в разделе ресурсов, используя идентификатор объекта и идентификатор определения роли:

```JSON
{
    "type": "Microsoft.Authorization/roleAssignments", 
    "apiVersion": "2020-04-01-preview",
    "name": "[parameters('vmIdentityRoleNameGuid')]",
    "scope": "[concat('Microsoft.ManagedIdentity/userAssignedIdentities', '/', parameters('userAssignedIdentityName'))]",
    "dependsOn": [ 
        "[concat('Microsoft.ManagedIdentity/userAssignedIdentities/', parameters('userAssignedIdentityName'))]"
    ], 
    "properties": {
        "roleDefinitionId": "[concat('/subscriptions/', subscription().subscriptionId, '/providers/Microsoft.Authorization/roleDefinitions/', 'f1a07417-d97a-45cb-824c-7a7467783830')]",
        "principalId": "fbc587f2-66f5-4459-a027-bcd908b9d278" 
    } 
}, 
```

или можно создать с помощью PowerShell, используя идентификатор приложения и идентификатор определения роли:

```powershell
New-AzRoleAssignment -ApplicationId 74cb6831-0dbb-4be1-8206-fd4df301cdc2 -RoleDefinitionName "Managed Identity Operator" -Scope "/subscriptions/<subscriptionId>/resourceGroups/<resourceGroupName>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<userAssignedIdentityName>"
```

Идентификатор объекта и идентификатор определения роли:

```powershell
New-AzRoleAssignment -PrincipalId fbc587f2-66f5-4459-a027-bcd908b9d278 -RoleDefinitionName "Managed Identity Operator" -Scope "/subscriptions/<subscriptionId>/resourceGroups/<resourceGroupName>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<userAssignedIdentityName>"
```

## <a name="add-managed-identity-properties-to-node-type-definition"></a>Добавление свойств управляемого удостоверения в определение типа узла

Наконец, добавьте `vmManagedIdentity` Свойства и `userAssignedIdentities` в определение типа узла управляемого кластера. Обязательно используйте **2021-01-01-Preview** или более поздней версии для `apiVersion` .

```json

 {
    "type": "Microsoft.ServiceFabric/managedclusters/nodetypes",
    "apiVersion": "2021-01-01-preview",
    ...
    "properties": {
        "isPrimary" : true,
        "vmInstanceCount": 5,
        "dataDiskSizeGB": 100,
        "vmSize": "Standard_D2_v2",
        "vmImagePublisher" : "MicrosoftWindowsServer",
        "vmImageOffer" : "WindowsServer",
        "vmImageSku" : "2019-Datacenter",
        "vmImageVersion" : "latest",
        "vmManagedIdentity": {
            "userAssignedIdentities": [
                "[resourceId('Microsoft.ManagedIdentity/userAssignedIdentities', parameters('userAssignedIdentityName'))]"
            ]
        }
    }
}
```

После развертывания созданное управляемое удостоверение было добавлено в масштабируемый набор виртуальных машин указанного типа узла и может использоваться как можно скорее, как и в любом неуправляемом кластере.

## <a name="troubleshooting"></a>Устранение неполадок

Неправильное Добавление назначения роли будет выполнено со следующей ошибкой в развертывании:

:::image type="content" source="media/how-to-managed-identity-managed-cluster-vmss/role-assignment-error.png" alt-text="Ошибка развертывания портал Azure, показывающая клиент с ИДЕНТИФИКАТОРом объекта или приложения SFRP, у которого нет разрешения на выполнение действия по управлению удостоверениями":::

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Развертывание приложения в управляемом кластере Service Fabric](./tutorial-managed-cluster-deploy-app.md)