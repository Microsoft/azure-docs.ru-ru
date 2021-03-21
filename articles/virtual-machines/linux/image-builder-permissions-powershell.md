---
title: Настройка разрешений службы Azure Image Builder с помощью PowerShell
description: Настройка требований для службы "Построитель образов виртуальных машин Azure", включая разрешения и привилегии с помощью PowerShell
author: danielsollondon
ms.author: danis
ms.date: 03/05/2021
ms.topic: article
ms.service: virtual-machines
ms.subservice: image-builder
ms.collection: linux
ms.openlocfilehash: 9f8793b6ea0ba454b66c525c2d53c1de2197d539
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102440213"
---
# <a name="configure-azure-image-builder-service-permissions-using-powershell"></a>Настройка разрешений службы Azure Image Builder с помощью PowerShell

При регистрации для (AIB) это предоставляет службе AIB разрешение на создание, удаление и администрирование промежуточной группы ресурсов (IT_ *) и имеет права на добавление в нее ресурсов, необходимых для сборки образа. Это делается с помощью имени субъекта-службы AIB, которое предоставляется в вашей подписке во время успешной регистрации.

Чтобы разрешить Конструктору образов виртуальных машин Azure распространять образы в управляемые образы или в Общую коллекцию образов, необходимо создать назначенное пользователем удостоверение Azure с разрешениями на чтение и запись образов. При доступе к службе хранилища Azure для чтения закрытых или открытых контейнеров потребуются разрешения.

Перед созданием образа необходимо настроить разрешения и привилегии. В следующих разделах подробно описано, как настроить возможные сценарии с помощью PowerShell.

> [!IMPORTANT]
> Конструктор образов Azure сейчас поддерживается в общедоступной предварительной версии.
> Эта предварительная версия предоставляется без соглашения об уровне обслуживания и не рекомендована для использования рабочей среде. Некоторые функции могут не поддерживаться или их возможности могут быть ограничены. Дополнительные сведения см. в статье [Дополнительные условия использования предварительных выпусков Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

## <a name="register-the-features"></a>Регистрация функций

Сначала необходимо зарегистрироваться для службы Azure Image Builder. Регистрация предоставляет службе разрешение на создание, Администрирование и удаление промежуточной группы ресурсов. Служба также имеет права на добавление ресурсов в группу, необходимую для сборки образа.

```powershell-interactive
Register-AzProviderFeature -FeatureName VirtualMachineTemplatePreview -ProviderNamespace Microsoft.VirtualMachineImages
```

## <a name="create-an-azure-user-assigned-managed-identity"></a>Создание управляемого удостоверения, назначенного пользователем Azure

Построитель образов Azure требует создания [управляемого удостоверения, назначенного пользователем Azure](../../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-cli.md). Построитель образов Azure использует управляемое пользователем удостоверение для чтения образов, записи образов и доступа к учетным записям хранения Azure. Вы предоставляете разрешение на идентификацию для действий с конкретными действиями в подписке.

> [!NOTE]
> Ранее в построителе образов Azure для предоставления разрешений группам ресурсов образов используется имя участника-службы (SPN) Azure Image Builder. Использование имени субъекта-службы будет нерекомендуемым. Вместо этого используйте управляемое пользователем удостоверение.

В следующем примере показано, как создать управляемое удостоверение, назначенное пользователем Azure. Замените параметры заполнителя, чтобы задать переменные.

| Параметр | Описание |
|---------|-------------|
| \<Resource group\> | Группа ресурсов, в которой создается управляемое пользователем удостоверение. |

```powershell-interactive
## Add AZ PS module to support AzUserAssignedIdentity
Install-Module -Name Az.ManagedServiceIdentity

$parameters = @{
    Name = 'aibIdentity'
    ResourceGroupName = '<Resource group>'
}
# create identity
New-AzUserAssignedIdentity @parameters
```

Дополнительные сведения о назначенных пользователем удостоверениях Azure см. в разделе о том, как создать удостоверение, в [управляемой пользователем документации по удостоверениям](../../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-cli.md) Azure.

## <a name="allow-image-builder-to-distribute-images"></a>Разрешить построитель образов распространять изображения

Чтобы в Azure Image Builder можно было распространять образы (управляемые образы или коллекция общих образов), службе "Построитель образов Azure" должна быть разрешено внедрять образы в эти группы ресурсов. Чтобы предоставить необходимые разрешения, необходимо создать назначаемое пользователем управляемое удостоверение и предоставить ему права доступа к группе ресурсов, в которой создается образ. Построитель образов Azure **не** имеет разрешения на доступ к ресурсам в других группах ресурсов в подписке. Необходимо выполнить явные действия, чтобы разрешить доступ, чтобы избежать сбоев сборок.

Для распространения образов не нужно предоставлять назначаемые пользователем права участника управляемого удостоверения в группе ресурсов. Однако управляемому удостоверению, назначенному пользователю, требуются следующие `Actions` разрешения Azure в группе ресурсов распространения:

```Actions
Microsoft.Compute/images/write
Microsoft.Compute/images/read
Microsoft.Compute/images/delete
```

При распространении в галерею общих образов вам также потребуется:

```Actions
Microsoft.Compute/galleries/read
Microsoft.Compute/galleries/images/read
Microsoft.Compute/galleries/images/versions/read
Microsoft.Compute/galleries/images/versions/write
```

## <a name="permission-to-customize-existing-images"></a>Разрешение на настройку существующих образов

Чтобы создать образы из исходных образов (управляемые образы или коллекция общих образов) для построителя образов Azure, службе "Построитель образов Azure" необходимо разрешить чтение образов в этих группах ресурсов. Чтобы предоставить необходимые разрешения, необходимо создать назначаемое пользователем управляемое удостоверение и предоставить ему права доступа к группе ресурсов, в которой находится образ.

Создание из существующего пользовательского образа:

```Actions
Microsoft.Compute/galleries/read
```

Создание из существующей версии коллекции общих образов:

```Actions
Microsoft.Compute/galleries/read
Microsoft.Compute/galleries/images/read
Microsoft.Compute/galleries/images/versions/read
```

## <a name="permission-to-customize-images-on-your-vnets"></a>Разрешение на настройку изображений в виртуальных сетей

В построителе образов Azure предусмотрена возможность развертывания и использования существующей виртуальной сети в подписке, что позволяет настраивать доступ к подключенным ресурсам.

Чтобы развернуть виртуальную машину в существующей виртуальной сети, не нужно предоставлять назначенные пользователю права участника управляемого удостоверения в группе ресурсов. Однако управляемому удостоверению, назначенному пользователю, требуются следующие `Actions` разрешения Azure для группы ресурсов виртуальной сети:

```Actions
Microsoft.Network/virtualNetworks/read
Microsoft.Network/virtualNetworks/subnets/join/action
```

## <a name="create-an-azure-role-definition"></a>Создание определения роли Azure

В следующих примерах создается определение роли Azure на основе действий, описанных в предыдущих разделах. Примеры применяются на уровне группы ресурсов. Оцените и протестируйте, если примеры достаточно детализированы для ваших требований. Для вашего сценария может потребоваться уточнить его до определенной коллекции общих образов.

Действия с изображением позволяют выполнять чтение и запись. Определите, что подходит для вашей среды. Например, создайте роль, позволяющую построителю образов Azure читать образы из группы ресурсов *example-RG-1* и записать образы в группу ресурсов *example-RG-2*.

### <a name="custom-image-azure-role-example"></a>Пример роли Azure пользовательского образа

В следующем примере создается роль Azure для использования и распространения исходного пользовательского образа. Затем вы предоставляете настраиваемой роли управляемому удостоверению, назначенному пользователем для построителя образов Azure.

Чтобы упростить замену значений в примере, сначала задайте следующие переменные. Замените параметры заполнителя, чтобы задать переменные.

| Параметр | Описание |
|---------|-------------|
| \<Subscription ID\> | идентификатор подписки Azure; |
| \<Resource group\> | Группа ресурсов для пользовательского образа |

```powershell-interactive
$sub_id = "<Subscription ID>"
# Resource group - For Preview, image builder will only support creating custom images in the same Resource Group as the source managed image.
$imageResourceGroup = "<Resource group>"
$identityName = "aibIdentity"

# Use a web request to download the sample JSON description
$sample_uri="https://raw.githubusercontent.com/azure/azvmimagebuilder/master/solutions/12_Creating_AIB_Security_Roles/aibRoleImageCreation.json"
$role_definition="aibRoleImageCreation.json"

Invoke-WebRequest -Uri $sample_uri -Outfile $role_definition -UseBasicParsing

# Create a unique role name to avoid clashes in the same Azure Active Directory domain
$timeInt=$(get-date -UFormat "%s")
$imageRoleDefName="Azure Image Builder Image Def"+$timeInt

# Update the JSON definition placeholders with variable values
((Get-Content -path $role_definition -Raw) -replace '<subscriptionID>',$sub_id) | Set-Content -Path $role_definition
((Get-Content -path $role_definition -Raw) -replace '<rgName>', $imageResourceGroup) | Set-Content -Path $role_definition
((Get-Content -path $role_definition -Raw) -replace 'Azure Image Builder Service Image Creation Role', $imageRoleDefName) | Set-Content -Path $role_definition

# Create a custom role from the aibRoleImageCreation.json description file. 
New-AzRoleDefinition -InputFile $role_definition

# Get the user-identity properties
$identityNameResourceId=$(Get-AzUserAssignedIdentity -ResourceGroupName $imageResourceGroup -Name $identityName).Id
$identityNamePrincipalId=$(Get-AzUserAssignedIdentity -ResourceGroupName $imageResourceGroup -Name $identityName).PrincipalId

# Grant the custom role to the user-assigned managed identity for Azure Image Builder.
$parameters = @{
    ObjectId = $identityNamePrincipalId
    RoleDefinitionName = $imageRoleDefName
    Scope = '/subscriptions/' + $sub_id + '/resourceGroups/' + $imageResourceGroup
}

New-AzRoleAssignment @parameters
```

### <a name="existing-vnet-azure-role-example"></a>Пример роли существующей виртуальной сети Azure

В следующем примере создается роль Azure для использования и распространения существующего образа виртуальной сети. Затем вы предоставляете настраиваемой роли управляемому удостоверению, назначенному пользователем для построителя образов Azure.

Чтобы упростить замену значений в примере, сначала задайте следующие переменные. Замените параметры заполнителя, чтобы задать переменные.

| Параметр | Описание |
|---------|-------------|
| \<Subscription ID\> | идентификатор подписки Azure; |
| \<Resource group\> | Группа ресурсов виртуальной сети |

```powershell-interactive
$sub_id = "<Subscription ID>"
$res_group = "<Resource group>"
$identityName = "aibIdentity"

# Use a web request to download the sample JSON description
$sample_uri="https://raw.githubusercontent.com/azure/azvmimagebuilder/master/solutions/12_Creating_AIB_Security_Roles/aibRoleNetworking.json"
$role_definition="aibRoleNetworking.json"

Invoke-WebRequest -Uri $sample_uri -Outfile $role_definition -UseBasicParsing

# Create a unique role name to avoid clashes in the same AAD domain
$timeInt=$(get-date -UFormat "%s")
$networkRoleDefName="Azure Image Builder Network Def"+$timeInt

# Update the JSON definition placeholders with variable values
((Get-Content -path $role_definition -Raw) -replace '<subscriptionID>',$sub_id) | Set-Content -Path $role_definition
((Get-Content -path $role_definition -Raw) -replace '<vnetRgName>', $res_group) | Set-Content -Path $role_definition
((Get-Content -path $role_definition -Raw) -replace 'Azure Image Builder Service Networking Role',$networkRoleDefName) | Set-Content -Path $role_definition

# Create a custom role from the aibRoleNetworking.json description file
New-AzRoleDefinition -InputFile $role_definition

# Get the user-identity properties
$identityNameResourceId=$(Get-AzUserAssignedIdentity -ResourceGroupName $imageResourceGroup -Name $identityName).Id
$identityNamePrincipalId=$(Get-AzUserAssignedIdentity -ResourceGroupName $imageResourceGroup -Name $identityName).PrincipalId

# Assign the custom role to the user-assigned managed identity for Azure Image Builder
$parameters = @{
    ObjectId = $identityNamePrincipalId
    RoleDefinitionName = $networkRoleDefName
    Scope = '/subscriptions/' + $sub_id + '/resourceGroups/' + $res_group
}

New-AzRoleAssignment @parameters
```

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения см. в разделе [Общие сведения о построителе образов Azure](../image-builder-overview.md).
