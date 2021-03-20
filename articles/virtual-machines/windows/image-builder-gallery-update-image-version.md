---
title: Создание новой версии образа из существующей версии образа с помощью Azure Image Builder (Предварительная версия)
description: Создание новой версии образа виртуальной машины из существующей версии образа с помощью Azure Image Builder в Windows.
author: cynthn
ms.author: cynthn
ms.date: 03/02/2021
ms.topic: how-to
ms.service: virtual-machines
ms.subervice: image-builder
ms.colletion: windows
ms.openlocfilehash: 0a53e8de8dd832e793ae12034c96ce9fe634ed7a
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101694110"
---
# <a name="preview-create-a-new-vm-image-version-from-an-existing-image-version-using-azure-image-builder-in-windows"></a>Предварительная версия. Создание новой версии образа виртуальной машины из существующей версии образа с помощью Azure Image Builder в Windows

В этой статье показано, как взять существующую версию образа в [коллекции общих образов](../shared-image-galleries.md), обновить ее и опубликовать в коллекции как новую версию образа.

Для настройки образа мы будем использовать простой шаблон JSON. JSON-файл, который мы используем: [helloImageTemplateforSIGfromWinSIG.json](https://raw.githubusercontent.com/azure/azvmimagebuilder/master/quickquickstarts/2_Creating_a_Custom_Win_Shared_Image_Gallery_Image_from_SIG/helloImageTemplateforSIGfromWinSIG.json). 

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

Если вы использовали [Создание образа и распространяете его в общую коллекцию образов](image-builder-gallery.md) для создания коллекции общих образов, вы уже создали нужные вам переменные. Если нет, настройте некоторые переменные для использования в этом примере.

В предварительной версии Конструктор образов поддерживает создание пользовательских образов только в той же группе ресурсов, где расположен исходный управляемый образ. Измените имя группы ресурсов, указанное в этом примере, чтобы оно совпадало с именем группы ресурсов для исходного управляемого образа.

```azurecli-interactive
# Resource group name - we are using ibsigRG in this example
sigResourceGroup=myIBWinRG
# Datacenter location - we are using West US 2 in this example
location=westus
# Additional region to replicate the image to - we are using East US in this example
additionalregion=eastus
# name of the shared image gallery - in this example we are using myGallery
sigName=my22stSIG
# name of the image definition to be created - in this example we are using myImageDef
imageDefName=winSvrimages
# image distribution metadata reference name
runOutputName=w2019SigRo
# User name and password for the VM
username="user name for the VM"
vmpassword="password for the VM"
```

Создайте переменную для идентификатора подписки. Его можно получить с помощью `az account show | grep id`.

```azurecli-interactive
subscriptionID=<Subscription ID>
```

Получение версии образа, которую требуется обновить.

```azurecli-interactive
sigDefImgVersionId=$(az sig image-version list \
   -g $sigResourceGroup \
   --gallery-name $sigName \
   --gallery-image-definition $imageDefName \
   --subscription $subscriptionID --query [].'id' -o json | grep 0. | tr -d '"' | tr -d '[:space:]')
```

## <a name="create-a-user-assigned-identity-and-set-permissions-on-the-resource-group"></a>Создание назначаемого пользователем удостоверения и задание разрешений для группы ресурсов
Так как вы установили удостоверение пользователя в предыдущем примере, вам нужно просто получить его идентификатор ресурса, а затем добавить его к шаблону.

```azurecli-interactive
#get identity used previously
imgBuilderId=$(az identity list -g $sigResourceGroup --query "[?contains(name, 'aibBuiUserId')].id" -o tsv)
```

Если у вас уже есть собственная коллекция общих образов и вы не следовали предыдущему примеру, вам нужно будет назначить разрешения для доступа к группе ресурсов для построителя изображений, чтобы она могла получить доступ к коллекции. Ознакомьтесь с примером в статье [Создание образа и распространение в коллекцию общих образов](image-builder-gallery.md) .


## <a name="modify-helloimage-example"></a>Пример изменения Хеллоимаже
Вы можете ознакомиться с примером, который мы будем использовать, открыв JSON файл здесь: [helloImageTemplateforSIGfromSIG.js](https://raw.githubusercontent.com/azure/azvmimagebuilder/master/quickquickstarts/2_Creating_a_Custom_Linux_Shared_Image_Gallery_Image_from_SIG/helloImageTemplateforSIGfromSIG.json) вместе с [ссылкой на шаблон построителя образов](../linux/image-builder-json.md). 


Скачайте пример JSON и настройте его с помощью переменных. 

```azurecli-interactive
curl https://raw.githubusercontent.com/azure/azvmimagebuilder/master/quickquickstarts/8_Creating_a_Custom_Win_Shared_Image_Gallery_Image_from_SIG/helloImageTemplateforSIGfromWinSIG.json -o helloImageTemplateforSIGfromWinSIG.json
sed -i -e "s/<subscriptionID>/$subscriptionID/g" helloImageTemplateforSIGfromWinSIG.json
sed -i -e "s/<rgName>/$sigResourceGroup/g" helloImageTemplateforSIGfromWinSIG.json
sed -i -e "s/<imageDefName>/$imageDefName/g" helloImageTemplateforSIGfromWinSIG.json
sed -i -e "s/<sharedImageGalName>/$sigName/g" helloImageTemplateforSIGfromWinSIG.json
sed -i -e "s%<sigDefImgVersionId>%$sigDefImgVersionId%g" helloImageTemplateforSIGfromWinSIG.json
sed -i -e "s/<region1>/$location/g" helloImageTemplateforSIGfromWinSIG.json
sed -i -e "s/<region2>/$additionalregion/g" helloImageTemplateforSIGfromWinSIG.json
sed -i -e "s/<runOutputName>/$runOutputName/g" helloImageTemplateforSIGfromWinSIG.json
sed -i -e "s%<imgBuilderId>%$imgBuilderId%g" helloImageTemplateforSIGfromWinSIG.json
```

## <a name="create-the-image"></a>Создание образа

Отправьте конфигурацию образа в службу "Построитель образов виртуальных машин".

```azurecli-interactive
az resource create \
    --resource-group $sigResourceGroup \
    --properties @helloImageTemplateforSIGfromWinSIG.json \
    --is-full-object \
    --resource-type Microsoft.VirtualMachineImages/imageTemplates \
    -n imageTemplateforSIGfromWinSIG01
```

Запустите сборку образа.

```azurecli-interactive
az resource invoke-action \
     --resource-group $sigResourceGroup \
     --resource-type  Microsoft.VirtualMachineImages/imageTemplates \
     -n imageTemplateforSIGfromWinSIG01 \
     --action Run 
```

Дождитесь завершения сборки и репликации образа, прежде чем переходить к следующему шагу.


## <a name="create-the-vm"></a>Создание виртуальной машины

```azurecli-interactive
az vm create \
  --resource-group $sigResourceGroup \
  --name aibImgWinVm002 \
  --admin-username $username \
  --admin-password $vmpassword \
  --image "/subscriptions/$subscriptionID/resourceGroups/$sigResourceGroup/providers/Microsoft.Compute/galleries/$sigName/images/$imageDefName/versions/latest" \
  --location $location
```

## <a name="verify-the-customization"></a>Проверка настройки
Подключитесь к виртуальной машине через подключение к удаленному рабочему столу, используя имя пользователя и пароль, заданные при создании виртуальной машины. В виртуальной машине откройте командную строку и введите следующую команду:

```console
dir c:\
```

Теперь вы увидите два каталога:
- `buildActions` , созданного в первой версии образа.
- `buildActions2` Это было создано как часть обновления первой версии образа для создания второй версии образа.


## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о компонентах JSON, использованных в этой статье, см. в разделе [Справочник по шаблонам для Image Builder](../linux/image-builder-json.md).
