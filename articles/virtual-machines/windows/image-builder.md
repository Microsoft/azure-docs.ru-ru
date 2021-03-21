---
title: Создание виртуальной машины Windows с помощью Azure Image Builder (Предварительная версия)
description: Создайте виртуальную машину Windows с помощью построителя образов Azure.
author: cynthn
ms.author: cynthn
ms.date: 03/02/2020
ms.topic: how-to
ms.service: virtual-machines
ms.subervice: image-builder
ms.colletion: windows
ms.openlocfilehash: 918cee723bfde69d08532aee6fe4f395dbddb4ee
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101695453"
---
# <a name="preview-create-a-windows-vm-with-azure-image-builder"></a>Предварительная версия: создание виртуальной машины Windows с помощью Azure Image Builder

В этой статье показано, как создать настраиваемый образ Windows с помощью построителя образов виртуальных машин Azure. В примере, приведенном в этой статье, используются [Настраиваемые](../linux/image-builder-json.md#properties-customize) для настройки изображения:
- PowerShell (Скриптури) — Скачайте и запустите [сценарий PowerShell](https://raw.githubusercontent.com/danielsollondon/azvmimagebuilder/master/testPsScript.ps1).
- Перезагрузка Windows — перезапускает виртуальную машину.
- PowerShell (встроенная) — выполнение определенной команды. В этом примере он создает каталог на виртуальной машине с помощью `mkdir c:\\buildActions` .
- Файл — копирование файла из GitHub на виртуальную машину. В этом примере [index.md](https://raw.githubusercontent.com/danielsollondon/azvmimagebuilder/master/quickquickstarts/exampleArtifacts/buildArtifacts/index.html) копируется на `c:\buildArtifacts\index.html` виртуальную машину.
- Буилдтимеаутинминутес — увеличение времени сборки для более длительного выполнения сборок, значение по умолчанию — 240 минут, и можно увеличить время сборки, чтобы обеспечить более длительное выполнение сборок.
- Вмпрофиле — указание свойств vmSize и сети
- Осдисксизегб — можно увеличить размер образа
- удостоверение. Предоставление удостоверения для построителя образов Azure для использования во время сборки


Можно также указать `buildTimeoutInMinutes` . Значение по умолчанию — 240 минут. Вы можете увеличить время сборки, чтобы обеспечить более длительное выполнение сборок.

Для настройки образа мы будем использовать простой шаблон JSON. JSON-файл, который мы используем: [helloImageTemplateWin.json](https://raw.githubusercontent.com/danielsollondon/azvmimagebuilder/master/quickquickstarts/0_Creating_a_Custom_Windows_Managed_Image/helloImageTemplateWin.json). 


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


## <a name="set-variables"></a>Задание переменных

Мы будем использовать несколько фрагментов информации повторно, поэтому создадим переменные для их хранения.


```azurecli-interactive
# Resource group name - we are using myImageBuilderRG in this example
imageResourceGroup=myWinImgBuilderRG
# Region location 
location=WestUS2
# Name for the image 
imageName=myWinBuilderImage
# Run output name
runOutputName=aibWindows
# name of the image to be created
imageName=aibWinImage
```

Создайте переменную для идентификатора подписки. Его можно получить с помощью `az account show | grep id`.

```azurecli-interactive
subscriptionID=<Your subscription ID>
```
## <a name="create-a-resource-group"></a>Создание группы ресурсов
Эта группа ресурсов используется для хранения артефакта шаблона конфигурации образа и образа.


```azurecli-interactive
az group create -n $imageResourceGroup -l $location
```

## <a name="create-a-user-assigned-identity-and-set-permissions-on-the-resource-group"></a>Создание назначаемого пользователем удостоверения и задание разрешений для группы ресурсов
Построитель образов будет использовать предоставленное [удостоверение пользователя](../../active-directory/managed-identities-azure-resources/qs-configure-cli-windows-vm.md#user-assigned-managed-identity) для внедрения образа в группу ресурсов. В этом примере вы создадите определение роли Azure с детализированными действиями для распространения образа. Затем определение роли будет назначено удостоверению пользователя.

## <a name="create-user-assigned-managed-identity-and-grant-permissions"></a>Создание управляемого удостоверения, назначаемого пользователем, и предоставление разрешений 
```bash
# create user assigned identity for image builder to access the storage account where the script is located
idenityName=aibBuiUserId$(date +'%s')
az identity create -g $imageResourceGroup -n $idenityName

# get identity id
imgBuilderCliId=$(az identity show -g $imageResourceGroup -n $idenityName | grep "clientId" | cut -c16- | tr -d '",')

# get the user identity URI, needed for the template
imgBuilderId=/subscriptions/$subscriptionID/resourcegroups/$imageResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/$idenityName

# download preconfigured role definition example
curl https://raw.githubusercontent.com/azure/azvmimagebuilder/master/solutions/12_Creating_AIB_Security_Roles/aibRoleImageCreation.json -o aibRoleImageCreation.json

imageRoleDefName="Azure Image Builder Image Def"$(date +'%s')

# update the definition
sed -i -e "s/<subscriptionID>/$subscriptionID/g" aibRoleImageCreation.json
sed -i -e "s/<rgName>/$imageResourceGroup/g" aibRoleImageCreation.json
sed -i -e "s/Azure Image Builder Service Image Creation Role/$imageRoleDefName/g" aibRoleImageCreation.json

# create role definitions
az role definition create --role-definition ./aibRoleImageCreation.json

# grant role definition to the user assigned identity
az role assignment create \
    --assignee $imgBuilderCliId \
    --role $imageRoleDefName \
    --scope /subscriptions/$subscriptionID/resourceGroups/$imageResourceGroup
```



## <a name="download-the-image-configuration-template-example"></a>Скачать пример шаблона конфигурации образа

Был создан шаблон конфигурации параметризованного образа, который вы хотите использовать. Скачайте файл example. JSON и настройте его с помощью ранее заданных переменных.

```azurecli-interactive
curl https://raw.githubusercontent.com/azure/azvmimagebuilder/master/quickquickstarts/0_Creating_a_Custom_Windows_Managed_Image/helloImageTemplateWin.json -o helloImageTemplateWin.json

sed -i -e "s/<subscriptionID>/$subscriptionID/g" helloImageTemplateWin.json
sed -i -e "s/<rgName>/$imageResourceGroup/g" helloImageTemplateWin.json
sed -i -e "s/<region>/$location/g" helloImageTemplateWin.json
sed -i -e "s/<imageName>/$imageName/g" helloImageTemplateWin.json
sed -i -e "s/<runOutputName>/$runOutputName/g" helloImageTemplateWin.json
sed -i -e "s%<imgBuilderId>%$imgBuilderId%g" helloImageTemplateWin.json

```

Этот пример можно изменить в терминале, используя текстовый редактор, например `vi` .

```azurecli-interactive
vi helloImageTemplateWin.json
```

> [!NOTE]
> Для исходного образа необходимо всегда [указывать версию](../linux/image-builder-troubleshoot.md#build--step-failed-for-image-version), но нельзя использовать `latest` .
> При добавлении или изменении группы ресурсов, в которую распространяется изображение, необходимо [установить разрешения](#create-a-user-assigned-identity-and-set-permissions-on-the-resource-group) для группы ресурсов.
 
## <a name="create-the-image"></a>Создание образа

Отправка конфигурации образа в службу "Построитель образов виртуальных машин"

```azurecli-interactive
az resource create \
    --resource-group $imageResourceGroup \
    --properties @helloImageTemplateWin.json \
    --is-full-object \
    --resource-type Microsoft.VirtualMachineImages/imageTemplates \
    -n helloImageTemplateWin01
```

По завершении это приведет к возврату сообщения об успешном выполнении в консоль и созданию `Image Builder Configuration Template` в `$imageResourceGroup` . Этот ресурс можно увидеть в группе ресурсов в портал Azure, если включить параметр "Показать скрытые типы".

В фоновом режиме построитель образов также создаст группу промежуточных ресурсов в подписке. Эта группа ресурсов используется для сборки образа. Он будет иметь следующий формат: `IT_<DestinationResourceGroup>_<TemplateName>`

> [!Note]
> Не следует удалять промежуточную группу ресурсов напрямую. Сначала удалите артефакт шаблона образа. это приведет к удалению промежуточной группы ресурсов.

Если служба сообщает о сбое во время отправки шаблона конфигурации образа:
-  Ознакомьтесь с этими действиями по [устранению неполадок](../linux/image-builder-troubleshoot.md#troubleshoot-image-template-submission-errors) . 
- Перед повторной попыткой отправки необходимо удалить шаблон с помощью следующего фрагмента кода.

```azurecli-interactive
az resource delete \
    --resource-group $imageResourceGroup \
    --resource-type Microsoft.VirtualMachineImages/imageTemplates \
    -n helloImageTemplateLinux01
```

## <a name="start-the-image-build"></a>Запуск сборки образа
Запустите процесс создания образа с помощью команды [AZ Resource Invoke-Action](/cli/azure/resource#az-resource-invoke-action).

```azurecli-interactive
az resource invoke-action \
     --resource-group $imageResourceGroup \
     --resource-type  Microsoft.VirtualMachineImages/imageTemplates \
     -n helloImageTemplateWin01 \
     --action Run 
```

Дождитесь завершения сборки. Это может занять около 15 минут.

При возникновении ошибок ознакомьтесь с этими действиями по [устранению неполадок](../linux/image-builder-troubleshoot.md#troubleshoot-common-build-errors) .


## <a name="create-the-vm"></a>Создание виртуальной машины

Создайте виртуальную машину с помощью созданного образа. Замените на свой *\<password>* собственный пароль `aibuser` на виртуальной машине.

```azurecli-interactive
az vm create \
  --resource-group $imageResourceGroup \
  --name aibImgWinVm00 \
  --admin-username aibuser \
  --admin-password <password> \
  --image $imageName \
  --location $location
```

## <a name="verify-the-customization"></a>Проверка настройки

Подключитесь к виртуальной машине через подключение к удаленному рабочему столу, используя имя пользователя и пароль, заданные при создании виртуальной машины. В виртуальной машине откройте командную строку и введите следующую команду:

```console
dir c:\
```

Вы должны увидеть эти два каталога, созданные во время настройки образа:
- буилдактионс
- буилдартифактс

## <a name="clean-up"></a>Очистка

По завершении удалите ресурсы.

### <a name="delete-the-image-builder-template"></a>Удаление шаблона построителя образов

```azurecli-interactive
az resource delete \
    --resource-group $imageResourceGroup \
    --resource-type Microsoft.VirtualMachineImages/imageTemplates \
    -n helloImageTemplateWin01
```

### <a name="delete-the-role-assignment-role-definition-and-user-identity"></a>Удалите назначение роли, определение роли и удостоверение пользователя.
```azurecli-interactive
az role assignment delete \
    --assignee $imgBuilderCliId \
    --role "$imageRoleDefName" \
    --scope /subscriptions/$subscriptionID/resourceGroups/$imageResourceGroup

az role definition delete --name "$imageRoleDefName"

az identity delete --ids $imgBuilderId
```

### <a name="delete-the-image-resource-group"></a>Удаление группы ресурсов образа

```azurecli-interactive
az group delete -n $imageResourceGroup
```


## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о компонентах JSON, использованных в этой статье, см. в разделе [Справочник по шаблонам для Image Builder](../linux/image-builder-json.md).
