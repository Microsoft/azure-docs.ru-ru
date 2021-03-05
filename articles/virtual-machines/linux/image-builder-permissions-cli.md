---
title: Настройка разрешений службы Azure Image Builder с помощью Azure CLI
description: Настройка требований для службы "Построитель образов виртуальных машин Azure", включая разрешения и привилегии, с помощью Azure CLI
author: cynthn
ms.author: danis
ms.date: 04/02/2021
ms.topic: article
ms.service: virtual-machines
ms.subservice: image-builder
ms.collection: linux
ms.openlocfilehash: 4b6154a18cf4e08bf59dad91350160a1f83c49ed
ms.sourcegitcommit: dda0d51d3d0e34d07faf231033d744ca4f2bbf4a
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102201487"
---
# <a name="configure-azure-image-builder-service-permissions-using-azure-cli"></a>Настройка разрешений службы Azure Image Builder с помощью Azure CLI

При регистрации для (AIB) это предоставляет службе AIB разрешение на создание, удаление и администрирование промежуточной группы ресурсов (IT_ *) и имеет права на добавление в нее ресурсов, необходимых для сборки образа. Это делается с помощью имени субъекта-службы AIB, которое предоставляется в вашей подписке во время успешной регистрации.

Чтобы разрешить Конструктору образов виртуальных машин Azure распространять образы в управляемые образы или в Общую коллекцию образов, необходимо создать назначенное пользователем удостоверение Azure с разрешениями на чтение и запись образов. При доступе к службе хранилища Azure для чтения закрытых или открытых контейнеров потребуются разрешения.

Перед созданием образа необходимо настроить разрешения и привилегии. В следующих разделах подробно описано, как настроить возможные сценарии с помощью Azure CLI.

> [!IMPORTANT]
> Конструктор образов Azure сейчас поддерживается в общедоступной предварительной версии.
> Эта предварительная версия предоставляется без соглашения об уровне обслуживания и не рекомендована для использования рабочей среде. Некоторые функции могут не поддерживаться или их возможности могут быть ограничены. Дополнительные сведения см. в статье [Дополнительные условия использования предварительных выпусков Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../includes/azure-cli-prepare-your-environment.md)]

## <a name="register-the-features"></a>Регистрация функций

Сначала необходимо зарегистрироваться для службы Azure Image Builder. Регистрация предоставляет службе разрешение на создание, Администрирование и удаление промежуточной группы ресурсов. Служба также имеет права на добавление ресурсов в группу, необходимую для сборки образа.

```azurecli-interactive
az feature register --namespace Microsoft.VirtualMachineImages --name VirtualMachineTemplatePreview
```

## <a name="create-an-azure-user-assigned-managed-identity"></a>Создание управляемого удостоверения, назначенного пользователем Azure

Построитель образов Azure требует создания [управляемого удостоверения, назначенного пользователем Azure](../../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-cli.md). Построитель образов Azure использует управляемое пользователем удостоверение для чтения образов, записи образов и доступа к учетным записям хранения Azure. Вы предоставляете разрешение на идентификацию для действий с конкретными действиями в подписке.

> [!NOTE]
> Ранее в построителе образов Azure для предоставления разрешений группам ресурсов образов используется имя участника-службы (SPN) Azure Image Builder. Использование имени субъекта-службы будет нерекомендуемым. Вместо этого используйте управляемое пользователем удостоверение.

В следующем примере показано, как создать управляемое удостоверение, назначенное пользователем Azure. Замените параметры заполнителя, чтобы задать переменные.

| Параметр | Описание |
|---------|-------------|
| \<Resource group\> | Группа ресурсов, в которой создается управляемое пользователем удостоверение. |

```azurecli-interactive
identityName="aibIdentity"
imageResourceGroup=<Resource group>

az identity create \
    --resource-group $imageResourceGroup \
    --name $identityName
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

```azurecli-interactive
# Subscription ID - You can get this using `az account show | grep id` or from the Azure portal.
subscriptionID=<Subscription ID>
# Resource group - For Preview, image builder will only support creating custom images in the same Resource Group as the source managed image.
imageResourceGroup=<Resource group>
identityName="aibIdentity"

# Use *cURL* to download the a sample JSON description 
curl https://raw.githubusercontent.com/azure/azvmimagebuilder/master/solutions/12_Creating_AIB_Security_Roles/aibRoleImageCreation.json -o aibRoleImageCreation.json

# Create a unique role name to avoid clashes in the same Azure Active Directory domain
imageRoleDefName="Azure Image Builder Image Def"$(date +'%s')

# Update the JSON definition using stream editor
sed -i -e "s/<subscriptionID>/$subscriptionID/g" aibRoleImageCreation.json
sed -i -e "s/<rgName>/$imageResourceGroup/g" aibRoleImageCreation.json
sed -i -e "s/Azure Image Builder Service Image Creation Role/$imageRoleDefName/g" aibRoleImageCreation.json

# Create a custom role from the sample aibRoleImageCreation.json description file.
az role definition create --role-definition ./aibRoleImageCreation.json

# Get the user-assigned managed identity id
imgBuilderCliId=$(az identity show -g $imageResourceGroup -n $identityName | grep "clientId" | cut -c16- | tr -d '",')

# Grant the custom role to the user-assigned managed identity for Azure Image Builder.
az role assignment create \
    --assignee $imgBuilderCliId \
    --role $imageRoleDefName \
    --scope /subscriptions/$subscriptionID/resourceGroups/$imageResourceGroup
```

### <a name="existing-vnet-azure-role-example"></a>Пример роли существующей виртуальной сети Azure

В следующем примере создается роль Azure для использования и распространения существующего образа виртуальной сети. Затем вы предоставляете настраиваемой роли управляемому удостоверению, назначенному пользователем для построителя образов Azure.

Чтобы упростить замену значений в примере, сначала задайте следующие переменные. Замените параметры заполнителя, чтобы задать переменные.

| Параметр | Описание |
|---------|-------------|
| \<Subscription ID\> | идентификатор подписки Azure; |
| \<Resource group\> | Группа ресурсов виртуальной сети |

```azurecli-interactive
# Subscription ID - You can get this using `az account show | grep id` or from the Azure portal.
subscriptionID=<Subscription ID>
VnetResourceGroup=<Resource group>
identityName="aibIdentity"

# Use *cURL* to download the a sample JSON description 
curl https://raw.githubusercontent.com/azure/azvmimagebuilder/master/solutions/12_Creating_AIB_Security_Roles/aibRoleNetworking.json -o aibRoleNetworking.json

# Create a unique role name to avoid clashes in the same domain
netRoleDefName="Azure Image Builder Network Def"$(date +'%s')

# Update the JSON definition using stream editor
sed -i -e "s/<subscriptionID>/$subscriptionID/g" aibRoleNetworking.json
sed -i -e "s/<vnetRgName>/$vnetRgName/g" aibRoleNetworking.json
sed -i -e "s/Azure Image Builder Service Networking Role/$netRoleDefName/g" aibRoleNetworking.json

# Create a custom role from the aibRoleNetworking.json description file.
az role definition create --role-definition ./aibRoleNetworking.json

# Get the user-assigned managed identity id
imgBuilderCliId=$(az identity show -g $imageResourceGroup -n $identityName | grep "clientId" | cut -c16- | tr -d '",')

# Grant the custom role to the user-assigned managed identity for Azure Image Builder.
az role assignment create \
    --assignee $imgBuilderCliId \
    --role $netRoleDefName \
    --scope /subscriptions/$subscriptionID/resourceGroups/$VnetResourceGroup
```

## <a name="using-managed-identity-for-azure-storage-access"></a>Использование управляемого удостоверения для доступа к хранилищу Azure

Если вы хотите симлессли проверку подлинности с помощью службы хранилища Azure и использовать частные контейнеры, Azure Image Builder вам потребуется назначенное пользователем управляемое удостоверение. Построитель образов Azure использует удостоверение для аутентификации в службе хранилища Azure.

> [!NOTE]
> Построитель образов Azure использует удостоверение только во время отправки шаблона образа. Виртуальная машина сборки не имеет доступа к удостоверению во время сборки образа.

Для создания управляемого удостоверения, назначенного пользователем, используйте Azure CLI.

```azurecli
az role assignment create \
    --assignee <Image Builder client ID> \
    --role "Storage Blob Data Reader" \
    --scope /subscriptions/<Subscription ID>/resourceGroups/<Resource group>/providers/Microsoft.Storage/storageAccounts/$scriptStorageAcc/blobServices/default/containers/<Storage account container>
```

В шаблоне построителя образов необходимо указать назначаемое пользователем управляемое удостоверение.

```json
    "type": "Microsoft.VirtualMachineImages/imageTemplates",
    "apiVersion": "2019-05-01-preview",
    "location": "<Region>",
    ..
    "identity": {
    "type": "UserAssigned",
          "userAssignedIdentities": {
            "<Image Builder ID>": {}     
        }
```

Замените следующие параметры заполнителя:

| Параметр | Описание |
|---------|-------------|
| \<Region\> | Область шаблона |
| \<Resource group\> | Группа ресурсов |
| \<Storage account container\> | имя контейнера в учетной записи хранения. |
| \<Subscription ID\> | Подписка Azure. |

Дополнительные сведения об управляемом удостоверении, назначенном пользователем, см. в разделе [Создание пользовательского образа, который будет использовать управляемое удостоверение User-Assigned Azure для доступа к файлам службы хранилища Azure](https://docs.microsoft.com/azure/virtual-machines/linux/image-builder-user-assigned-identity). Краткое руководство посвящено созданию и настройке управляемого удостоверения, назначаемого пользователем, для доступа к учетной записи хранения.

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения см. в разделе [Общие сведения о построителе образов Azure](../image-builder-overview.md).
