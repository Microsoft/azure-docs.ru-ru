---
title: Использование Azure Image Builder & коллекции общих образов для виртуальных машин Linux (Предварительная версия)
description: Узнайте, как использовать построитель образов Azure и Azure CLI, чтобы создать версию образа в общей коллекции образов, а затем распространить образ глобально.
author: cynthn
ms.author: cynthn
ms.date: 03/02/2021
ms.topic: how-to
ms.service: virtual-machines
ms.subservice: image-builder
ms.collection: linux
ms.reviewer: danis
ms.openlocfilehash: cf7f5ed4a27772a89d0356c60e6f7135786ca07d
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101695606"
---
# <a name="preview-create-a-linux-image-and-distribute-it-to-a-shared-image-gallery"></a>Предварительный просмотр: Создание образа Linux и его распространение в Общей коллекции образов 

В этой статье описано, как можно использовать Конструктор образов Azure и Azure CLI для создания версии образа в [Общей коллекции образов](../shared-image-galleries.md) и глобального распространения этого образа. Это также можно сделать [с помощью Azure PowerShell](../windows/image-builder-gallery.md).


Для настройки образа мы будем использовать простой шаблон JSON. JSON-файл, который мы используем, доступен здесь: [helloImageTemplateforSIG.json](https://github.com/azure/azvmimagebuilder/blob/master/quickquickstarts/1_Creating_a_Custom_Linux_Shared_Image_Gallery_Image/helloImageTemplateforSIG.json). 

С целью распространения образа в Общей коллекции образов для раздела шаблона `distribute` используется значение [sharedImage](image-builder-json.md#distribute-sharedimage).

> [!IMPORTANT]
> Конструктор образов Azure сейчас поддерживается в общедоступной предварительной версии.
> Эта предварительная версия предоставляется без соглашения об уровне обслуживания и не рекомендована для использования рабочей среде. Некоторые функции могут не поддерживаться или их возможности могут быть ограничены. Дополнительные сведения см. в статье [Дополнительные условия использования предварительных выпусков Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="register-the-features"></a>Регистрация функций
Чтобы использовать Конструктор образов Azure на этапе предварительной версии, необходимо зарегистрировать новую функцию.

```azurecli-interactive
az feature register --namespace Microsoft.VirtualMachineImages --name VirtualMachineTemplatePreview
```

Проверьте состояние регистрации функции.

```azurecli-interactive
az feature show --namespace Microsoft.VirtualMachineImages --name VirtualMachineTemplatePreview | grep state
```

Проверьте свою регистрацию.

```azurecli-interactive
az provider show -n Microsoft.VirtualMachineImages | grep registrationState
az provider show -n Microsoft.KeyVault | grep registrationState
az provider show -n Microsoft.Compute | grep registrationState
az provider show -n Microsoft.Storage | grep registrationState
az provider show -n Microsoft.Network | grep registrationState
```

Если регистрация не выполнена, примените следующую команду:

```azurecli-interactive
az provider register -n Microsoft.VirtualMachineImages
az provider register -n Microsoft.Compute
az provider register -n Microsoft.KeyVault
az provider register -n Microsoft.Storage
az provider register -n Microsoft.Network
```

## <a name="set-variables-and-permissions"></a>Настройка переменных и разрешений 

Мы будем использовать несколько фрагментов информации повторно, поэтому создадим переменные для их хранения.

В предварительной версии Конструктор образов поддерживает создание пользовательских образов только в той же группе ресурсов, где расположен исходный управляемый образ. Измените имя группы ресурсов, указанное в этом примере, чтобы оно совпадало с именем группы ресурсов для исходного управляемого образа.

```azurecli-interactive
# Resource group name - we are using ibLinuxGalleryRG in this example
sigResourceGroup=ibLinuxGalleryRG
# Datacenter location - we are using West US 2 in this example
location=westus2
# Additional region to replicate the image to - we are using East US in this example
additionalregion=eastus
# name of the shared image gallery - in this example we are using myGallery
sigName=myIbGallery
# name of the image definition to be created - in this example we are using myImageDef
imageDefName=myIbImageDef
# image distribution metadata reference name
runOutputName=aibLinuxSIG
```

Создайте переменную для идентификатора подписки. Его можно получить с помощью `az account show | grep id`.

```azurecli-interactive
subscriptionID=<Subscription ID>
```

Создайте группу ресурсов.

```azurecli-interactive
az group create -n $sigResourceGroup -l $location
```

## <a name="create-a-user-assigned-identity-and-set-permissions-on-the-resource-group"></a>Создание назначаемого пользователем удостоверения и задание разрешений для группы ресурсов
Для вставки образа в Общую коллекцию образов Azure Конструктор образов будет использовать предоставленное [удостоверение пользователя](../../active-directory/managed-identities-azure-resources/qs-configure-cli-windows-vm.md#user-assigned-managed-identity). В этом примере вы создадите определение роли Azure с детализированными действиями для распространения образа в Общую коллекцию образов. Затем определение роли будет назначено удостоверению пользователя.

```bash
# create user assigned identity for image builder to access the storage account where the script is located
idenityName=aibBuiUserId$(date +'%s')
az identity create -g $sigResourceGroup -n $idenityName

# get identity id
imgBuilderCliId=$(az identity show -g $sigResourceGroup -n $idenityName | grep "clientId" | cut -c16- | tr -d '",')

# get the user identity URI, needed for the template
imgBuilderId=/subscriptions/$subscriptionID/resourcegroups/$sigResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/$idenityName

# this command will download an Azure role definition template, and update the template with the parameters specified earlier.
curl https://raw.githubusercontent.com/azure/azvmimagebuilder/master/solutions/12_Creating_AIB_Security_Roles/aibRoleImageCreation.json -o aibRoleImageCreation.json

imageRoleDefName="Azure Image Builder Image Def"$(date +'%s')

# update the definition
sed -i -e "s/<subscriptionID>/$subscriptionID/g" aibRoleImageCreation.json
sed -i -e "s/<rgName>/$sigResourceGroup/g" aibRoleImageCreation.json
sed -i -e "s/Azure Image Builder Service Image Creation Role/$imageRoleDefName/g" aibRoleImageCreation.json

# create role definitions
az role definition create --role-definition ./aibRoleImageCreation.json

# grant role definition to the user assigned identity
az role assignment create \
    --assignee $imgBuilderCliId \
    --role $imageRoleDefName \
    --scope /subscriptions/$subscriptionID/resourceGroups/$sigResourceGroup
```


## <a name="create-an-image-definition-and-gallery"></a>Создание определения образа и коллекции

Чтобы использовать Конструктор образов с Общей коллекцией образов, вам потребуются существующая коллекция образов и определение образа. Конструктор образов не будет самостоятельно создавать коллекцию образов и определение образа.

Если у вас еще нет коллекции и определения образа, начните с их создания. Сначала создайте коллекцию образов.

```azurecli-interactive
az sig create \
    -g $sigResourceGroup \
    --gallery-name $sigName
```

Затем создайте определение образа.

```azurecli-interactive
az sig image-definition create \
   -g $sigResourceGroup \
   --gallery-name $sigName \
   --gallery-image-definition $imageDefName \
   --publisher myIbPublisher \
   --offer myOffer \
   --sku 18.04-LTS \
   --os-type Linux
```


## <a name="download-and-configure-the-json"></a>Скачивание и настройка JSON-файла

Скачайте JSON-файл шаблона и настройте его, указав нужные значения переменных.

```azurecli-interactive
curl https://raw.githubusercontent.com/azure/azvmimagebuilder/master/quickquickstarts/1_Creating_a_Custom_Linux_Shared_Image_Gallery_Image/helloImageTemplateforSIG.json -o helloImageTemplateforSIG.json
sed -i -e "s/<subscriptionID>/$subscriptionID/g" helloImageTemplateforSIG.json
sed -i -e "s/<rgName>/$sigResourceGroup/g" helloImageTemplateforSIG.json
sed -i -e "s/<imageDefName>/$imageDefName/g" helloImageTemplateforSIG.json
sed -i -e "s/<sharedImageGalName>/$sigName/g" helloImageTemplateforSIG.json
sed -i -e "s/<region1>/$location/g" helloImageTemplateforSIG.json
sed -i -e "s/<region2>/$additionalregion/g" helloImageTemplateforSIG.json
sed -i -e "s/<runOutputName>/$runOutputName/g" helloImageTemplateforSIG.json
sed -i -e "s%<imgBuilderId>%$imgBuilderId%g" helloImageTemplateforSIG.json
```

## <a name="create-the-image-version"></a>Создание версии образа

В следующей части будет создана версия образа в коллекции. 

Отправьте конфигурацию образа в службу "Конструктор образов Azure".

```azurecli-interactive
az resource create \
    --resource-group $sigResourceGroup \
    --properties @helloImageTemplateforSIG.json \
    --is-full-object \
    --resource-type Microsoft.VirtualMachineImages/imageTemplates \
    -n helloImageTemplateforSIG01
```

Запустите сборку образа.

```azurecli-interactive
az resource invoke-action \
     --resource-group $sigResourceGroup \
     --resource-type  Microsoft.VirtualMachineImages/imageTemplates \
     -n helloImageTemplateforSIG01 \
     --action Run 
```

Создание образа и его репликация в оба региона могут занять некоторое время. Дождитесь завершения этих процессов, прежде чем переходить к созданию виртуальной машины.


## <a name="create-the-vm"></a>Создание виртуальной машины

Создайте виртуальную машину на основе версии образа, созданной Конструктором образов Azure.

```azurecli-interactive
az vm create \
  --resource-group $sigResourceGroup \
  --name myAibGalleryVM \
  --admin-username aibuser \
  --location $location \
  --image "/subscriptions/$subscriptionID/resourceGroups/$sigResourceGroup/providers/Microsoft.Compute/galleries/$sigName/images/$imageDefName/versions/latest" \
  --generate-ssh-keys
```

Установите SSH-подключение к виртуальной машине.

```azurecli-interactive
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

Если вы хотите повторно настроить версию образа, чтобы создать новую версию того же образа, пропустите этот шаг и перейдите к статье об [использовании Конструктора образов Azure для создания другой версии образа](image-builder-gallery-update-image-version.md).


Это приведет к удалению созданного образа вместе со всеми остальными файлами ресурсов. Убедитесь, что вы завершили работу с этим развертыванием, прежде чем удалять ресурсы.

При удалении ресурсов для коллекции образов вам придется удалить все версии образа, прежде чем удалять определение образа, использованное для их создания. Чтобы удалить коллекцию, сначала необходимо удалить все определения образов в коллекции.

Удалите шаблон конструктора образов.

```azurecli-interactive
az resource delete \
    --resource-group $sigResourceGroup \
    --resource-type Microsoft.VirtualMachineImages/imageTemplates \
    -n helloImageTemplateforSIG01
```

Удалите назначения разрешений, роли и удостоверение.
```azurecli-interactive
az role assignment delete \
    --assignee $imgBuilderCliId \
    --role "$imageRoleDefName" \
    --scope /subscriptions/$subscriptionID/resourceGroups/$sigResourceGroup

az role definition delete --name "$imageRoleDefName"

az identity delete --ids $imgBuilderId
```

Получите версию образа (всегда начинается с префикса `0.`), созданную в Конструкторе образов, а затем удалите версию образа.

```azurecli-interactive
sigDefImgVersion=$(az sig image-version list \
   -g $sigResourceGroup \
   --gallery-name $sigName \
   --gallery-image-definition $imageDefName \
   --subscription $subscriptionID --query [].'name' -o json | grep 0. | tr -d '"')
az sig image-version delete \
   -g $sigResourceGroup \
   --gallery-image-version $sigDefImgVersion \
   --gallery-name $sigName \
   --gallery-image-definition $imageDefName \
   --subscription $subscriptionID
```   


Удалите определение образа.

```azurecli-interactive
az sig image-definition delete \
   -g $sigResourceGroup \
   --gallery-name $sigName \
   --gallery-image-definition $imageDefName \
   --subscription $subscriptionID
```

Удалите коллекцию.

```azurecli-interactive
az sig delete -r $sigName -g $sigResourceGroup
```

Удалите ее.

```azurecli-interactive
az group delete -n $sigResourceGroup -y
```

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения см. в статье об [Общих коллекциях образов Azure](../shared-image-galleries.md).
