---
title: Использование Azure Image Builder для виртуальных машин Linux, разрешающих доступ к существующей виртуальной сети Azure (Предварительная версия)
description: Создание образов виртуальных машин Linux с помощью Azure Image Builder, разрешающей доступ к существующей виртуальной сети Azure
author: danielsollondon
ms.author: danis
ms.date: 03/02/2021
ms.topic: how-to
ms.service: virtual-machines
ms.subservice: image-builder
ms.collection: linux
ms.reviewer: danis
ms.openlocfilehash: 500ddec9b84f9d73db45ddb4b7f5a8486a48d3e5
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102565314"
---
# <a name="use-azure-image-builder-for-linux-vms-allowing-access-to-an-existing-azure-vnet"></a>Использование Azure Image Builder для виртуальных машин Linux, разрешающих доступ к существующей виртуальной сети Azure

В этой статье показано, как с помощью построителя образов Azure создать базовый настроенный образ Linux, имеющий доступ к существующим ресурсам в виртуальной сети. Созданная виртуальная машина сборки развертывается в новой или существующей виртуальной сети, указанной в подписке. При использовании существующей ВИРТУАЛЬНОЙ сети Azure службе "Построитель образов Azure" не требуется подключение к общедоступному сетевому интерфейсу.

> [!IMPORTANT]
> Конструктор образов Azure сейчас поддерживается в общедоступной предварительной версии.
> Эта предварительная версия предоставляется без соглашения об уровне обслуживания и не рекомендована для использования рабочей среде. Некоторые функции могут не поддерживаться или их возможности могут быть ограничены. Дополнительные сведения см. в статье [Дополнительные условия использования предварительных выпусков Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../includes/azure-cli-prepare-your-environment.md)]

## <a name="register-the-features"></a>Регистрация функций

Сначала необходимо зарегистрироваться для службы Azure Image Builder. Регистрация предоставляет службе разрешение на создание, Администрирование и удаление промежуточной группы ресурсов. Служба также имеет права на добавление ресурсов в группу, необходимую для сборки образа.

```azurecli-interactive
az feature register --namespace Microsoft.VirtualMachineImages --name VirtualMachineTemplatePreview
```

## <a name="set-variables-and-permissions"></a>Настройка переменных и разрешений 

Несколько фрагментов информации будут многократно использоваться. Создайте переменные для хранения этих сведений.

```azurecli-interactive
# set your environment variables here!!!!

# destination image resource group
imageResourceGroup=aibImageRG01

# location (see possible locations in main docs)
location=WestUS2

# your subscription
# get the current subID : 'az account show | grep id'
subscriptionID=$(az account show | grep id | tr -d '",' | cut -c7-)

# name of the image to be created
imageName=aibCustomLinuxImg01

# image distribution metadata reference name
runOutputName=aibCustLinManImg01ro


# VNET properties (update to match your existing VNET, or leave as-is for demo)
# VNET name
vnetName=myexistingvnet01
# subnet name
subnetName=subnet01
# VNET resource group name
# NOTE! The VNET must always be in the same region as the AIB service region.
vnetRgName=existingVnetRG
# Existing Subnet NSG Name or the demo will create it
nsgName=aibdemoNsg
```

Создание группы ресурсов.

```azurecli-interactive
az group create -n $imageResourceGroup -l $location
```

## <a name="configure-networking"></a>Настройка сети

Если у вас нет существующего Внет\субнет\нсг, используйте следующий скрипт, чтобы создать его.

```bash

# Create a resource group

az group create -n $vnetRgName -l $location

# Create VNET

az network vnet create \
    --resource-group $vnetRgName \
    --name $vnetName --address-prefix 10.0.0.0/16 \
    --subnet-name $subnetName --subnet-prefix 10.0.0.0/24

# Create base NSG to simulate an existing NSG

az network nsg create -g $vnetRgName -n $nsgName

az network vnet subnet update \
    --resource-group $vnetRgName \
    --vnet-name $vnetName \
    --name $subnetName \
    --network-security-group $nsgName
    
#  NOTE! The VNET must always be in the same region as the Azure Image Builder service region.
```

### <a name="add-network-security-group-rule"></a>Добавить правило группы безопасности сети

Это правило разрешает подключение из балансировщика нагрузки построителя образов Azure к виртуальной машине прокси-сервера. Порт 60001 предназначен для ОС Linux и порт 60000 для ОС Windows. Прокси-сервер виртуальной машины подключается к виртуальной машине сборки, используя порт 22 для ОС Linux или порт 5986 для ОС Windows.

```azurecli-interactive
az network nsg rule create \
    --resource-group $vnetRgName \
    --nsg-name $nsgName \
    -n AzureImageBuilderNsgRule \
    --priority 400 \
    --source-address-prefixes AzureLoadBalancer \
    --destination-address-prefixes VirtualNetwork \
    --destination-port-ranges 60000-60001 --direction inbound \
    --access Allow --protocol Tcp \
    --description "Allow Image Builder Private Link Access to Proxy VM"
```

### <a name="disable-private-service-policy-on-subnet"></a>Отключить политику частной службы в подсети

```azurecli-interactive
az network vnet subnet update \
  --name $subnetName \
  --resource-group $vnetRgName \
  --vnet-name $vnetName \
  --disable-private-link-service-network-policies true 
```

Дополнительные сведения о сетях с построителем образов см. в статье [Сетевые параметры службы Azure Image Builder](image-builder-networking.md).

## <a name="modify-the-example-template-and-create-role"></a>Изменение примера шаблона и создание роли

```bash
# download the example and configure it with your vars

curl https://raw.githubusercontent.com/azure/azvmimagebuilder/master/quickquickstarts/1a_Creating_a_Custom_Linux_Image_on_Existing_VNET/existingVNETLinux.json -o existingVNETLinux.json
curl https://raw.githubusercontent.com/azure/azvmimagebuilder/master/solutions/12_Creating_AIB_Security_Roles/aibRoleNetworking.json -o aibRoleNetworking.json
curl https://raw.githubusercontent.com/azure/azvmimagebuilder/master/solutions/12_Creating_AIB_Security_Roles/aibRoleImageCreation.json -o aibRoleImageCreation.json

sed -i -e "s/<subscriptionID>/$subscriptionID/g" existingVNETLinux.json
sed -i -e "s/<rgName>/$imageResourceGroup/g" existingVNETLinux.json
sed -i -e "s/<region>/$location/g" existingVNETLinux.json
sed -i -e "s/<imageName>/$imageName/g" existingVNETLinux.json
sed -i -e "s/<runOutputName>/$runOutputName/g" existingVNETLinux.json

sed -i -e "s/<vnetName>/$vnetName/g" existingVNETLinux.json
sed -i -e "s/<subnetName>/$subnetName/g" existingVNETLinux.json
sed -i -e "s/<vnetRgName>/$vnetRgName/g" existingVNETLinux.json

sed -i -e "s/<subscriptionID>/$subscriptionID/g" aibRoleImageCreation.json
sed -i -e "s/<rgName>/$imageResourceGroup/g" aibRoleImageCreation.json

sed -i -e "s/<subscriptionID>/$subscriptionID/g" aibRoleNetworking.json
sed -i -e "s/<vnetRgName>/$vnetRgName/g" aibRoleNetworking.json

```

## <a name="set-permissions-on-the-resource-group"></a>Задание разрешений для группы ресурсов

Для вставки образа в Общую коллекцию образов Azure Конструктор образов будет использовать предоставленное [удостоверение пользователя](../../active-directory/managed-identities-azure-resources/qs-configure-cli-windows-vm.md#user-assigned-managed-identity). В этом примере вы создадите определение роли Azure с детализированными действиями для распространения образа в Общую коллекцию образов. Затем определение роли будет назначено удостоверению пользователя.

```bash
# create user assigned identity for image builder
idenityName=aibBuiUserId$(date +'%s')
az identity create -g $imageResourceGroup -n $idenityName

# get identity id
imgBuilderCliId=$(az identity show -g $imageResourceGroup -n $idenityName | grep "clientId" | cut -c16- | tr -d '",')

# get the user identity URI, needed for the template
imgBuilderId=/subscriptions/$subscriptionID/resourcegroups/$imageResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/$idenityName

# update the template
sed -i -e "s%<imgBuilderId>%$imgBuilderId%g" existingVNETLinux.json

# make role name unique, to avoid clashes in the same Azure Active Directory domain
imageRoleDefName="Azure Image Builder Image Def"$(date +'%s')
netRoleDefName="Azure Image Builder Network Def"$(date +'%s')

# update the definitions
sed -i -e "s/Azure Image Builder Service Image Creation Role/$imageRoleDefName/g" aibRoleImageCreation.json
sed -i -e "s/Azure Image Builder Service Networking Role/$netRoleDefName/g" aibRoleNetworking.json
```

Вместо предоставления более низкого уровня детализации и привилегий для построителя образов можно создать две роли. Один предоставляет построителю разрешения на создание образа, а другой позволяет ему подключить виртуальную машину сборки и подсистему балансировки нагрузки к виртуальной сети.

```bash
# create role definitions
az role definition create --role-definition ./aibRoleImageCreation.json
az role definition create --role-definition ./aibRoleNetworking.json

# grant role definition to the user assigned identity
az role assignment create \
    --assignee $imgBuilderCliId \
    --role $imageRoleDefName \
    --scope /subscriptions/$subscriptionID/resourceGroups/$imageResourceGroup

az role assignment create \
    --assignee $imgBuilderCliId \
    --role $netRoleDefName \
    --scope /subscriptions/$subscriptionID/resourceGroups/$vnetRgName
```

Дополнительные сведения о разрешениях см. в статье [Настройка разрешений службы Azure Image Builder с помощью Azure CLI](image-builder-permissions-cli.md) или [Настройка разрешений службы Azure Image Builder с помощью PowerShell](image-builder-permissions-powershell.md).

## <a name="create-the-image"></a>Создание образа

Отправьте конфигурацию образа в службу "Конструктор образов Azure".

```azurecli-interactive
az resource create \
    --resource-group $imageResourceGroup \
    --properties @existingVNETLinux.json \
    --is-full-object \
    --resource-type Microsoft.VirtualMachineImages/imageTemplates \
    -n existingVNETLinuxTemplate01

# Wait approximately 1-3 mins (validation, permissions etc.)
```

Запустите сборку образа.

```azurecli-interactive
az resource invoke-action \
     --resource-group $imageResourceGroup \
     --resource-type  Microsoft.VirtualMachineImages/imageTemplates \
     -n existingVNETLinuxTemplate01 \
     --action Run 

# Wait approximately 15 mins
```

Создание образа и его репликация в оба региона могут занять некоторое время. Дождитесь завершения этих процессов, прежде чем переходить к созданию виртуальной машины.


## <a name="create-the-vm"></a>Создание виртуальной машины

Создайте виртуальную машину на основе версии образа, созданной Конструктором образов Azure.

```azurecli-interactive
az vm create \
  --resource-group $imageResourceGroup \
  --name aibImgVm0001 \
  --admin-username aibuser \
  --image $imageName \
  --location $location \
  --generate-ssh-keys
```

Установите SSH-подключение к виртуальной машине.

```bash
ssh aibuser@<publicIpAddress>
```

Вы увидите, что для образа настроено *сообщение дня*, как только SSH-подключение будет установлено.

```console
*******************************************************
**            This VM was built from the:            **
**      !! AZURE VM IMAGE BUILDER Custom Image !!    **
**         You have just been Customized :-)         **
*******************************************************
```

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы хотите выполнить повторную настройку версии образа, чтобы создать новую версию того же образа, пропустите дальнейшие действия и перейдите к разделу [Использование Azure Image Builder для создания другой версии образа](image-builder-gallery-update-image-version.md).


После этого будет удален созданный образ вместе со всеми другими файлами ресурсов. Убедитесь, что вы завершили работу с этим развертыванием, прежде чем удалять ресурсы.

При удалении ресурсов для коллекции образов вам придется удалить все версии образа, прежде чем удалять определение образа, использованное для их создания. Чтобы удалить коллекцию, сначала необходимо удалить все определения образов в коллекции.

Удалите шаблон конструктора образов.

```azurecli
az resource delete \
    --resource-group $imageResourceGroup \
    --resource-type Microsoft.VirtualMachineImages/imageTemplates \
    -n existingVNETLinuxTemplate01
```

Удаление разрешений, ролей и удостоверений
```azurecli-interactive
az role assignment delete \
    --assignee $imgBuilderCliId \
    --role $imageRoleDefName \
    --scope /subscriptions/$subscriptionID/resourceGroups/$imageResourceGroup

az role assignment delete \
    --assignee $imgBuilderCliId \
    --role $netRoleDefName \
    --scope /subscriptions/$subscriptionID/resourceGroups/$vnetRgName


az role definition delete --name "$imageRoleDefName"
az role definition delete --name "$netRoleDefName"

az identity delete --ids $imgBuilderId
```

Удалите ее.

```azurecli-interactive
az group delete -n $imageResourceGroup
```

Если вы создали виртуальную сеть для этого краткого руководства, вы можете удалить виртуальную сеть, если она больше не используется.

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения см. в статье об [Общих коллекциях образов Azure](../shared-image-galleries.md).
