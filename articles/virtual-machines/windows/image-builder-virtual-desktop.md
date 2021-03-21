---
title: Построитель образов — создание образа виртуального рабочего стола Windows
description: Создайте образ виртуальной машины Azure для виртуальных рабочих столов Windows с помощью Azure Image Builder в PowerShell.
author: danielsollondon
ms.author: danis
ms.reviewer: cynthn
ms.date: 01/27/2021
ms.topic: article
ms.service: virtual-machines-windows
ms.collection: windows
ms.subservice: imaging
ms.openlocfilehash: 01b253747791fc29abf4434bebfd85865099f9ee
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "103602024"
---
# <a name="create-a-windows-virtual-desktop-image-using-azure-vm-image-builder-and-powershell"></a>Создание образа виртуального рабочего стола Windows с помощью построителя образов виртуальных машин Azure и PowerShell

В этой статье показано, как создать образ виртуального рабочего стола Windows с этими настройками:

* Установка [фслогикс](https://github.com/DeanCefola/Azure-WVD/blob/master/PowerShell/FSLogixSetup.ps1).
* Запуск [скрипта оптимизации виртуальных рабочих столов Windows](https://github.com/The-Virtual-Desktop-Team/Virtual-Desktop-Optimization-Tool) из репозитория сообщества.
* Установите [Microsoft Teams](https://docs.microsoft.com/azure/virtual-desktop/teams-on-wvd).
* [Перезапуск](https://docs.microsoft.com/azure/virtual-machines/linux/image-builder-json?toc=%2Fazure%2Fvirtual-machines%2Fwindows%2Ftoc.json&bc=%2Fazure%2Fvirtual-machines%2Fwindows%2Fbreadcrumb%2Ftoc.json#windows-restart-customizer)
* Запустить [Центр обновления Windows](https://docs.microsoft.com/azure/virtual-machines/linux/image-builder-json?toc=%2Fazure%2Fvirtual-machines%2Fwindows%2Ftoc.json&bc=%2Fazure%2Fvirtual-machines%2Fwindows%2Fbreadcrumb%2Ftoc.json#windows-update-customizer)

Мы покажем, как автоматизировать это с помощью построителя образов виртуальных машин Azure, и распространите образ в [общую коллекцию образов](https://docs.microsoft.com/azure/virtual-machines/windows/shared-image-galleries), где можно выполнять репликацию в другие регионы, управлять масштабом и совместно использовать образ внутри и за пределами Организации.


Для упрощения развертывания конфигурации построителя образов в этом примере используется шаблон Azure Resource Manager с шаблоном построителя изображений, вложенным в. Это дает некоторые другие преимущества, такие как переменные и входы параметров. Кроме того, можно передавать параметры из командной строки.

Эта статья предназначена для упражнений по копированию и вставке.

> [!NOTE]
> Скрипты для установки приложений находятся на сайте [GitHub](https://github.com/danielsollondon/azvmimagebuilder/tree/master/solutions/14_Building_Images_WVD). Они предназначены только для иллюстрации и тестирования, а не для рабочих нагрузок. 

## <a name="tips-for-building-windows-images"></a>Советы по созданию образов Windows 

- Размер виртуальной машины. размер виртуальной машины по умолчанию — `Standard_D1_v2` , который не подходит для Windows. Используйте `Standard_D2_v2` или более.
- В этом примере используются [скрипты настраиваемого PowerShell](../linux/image-builder-json.md). Необходимо использовать эти параметры, иначе сборка перестанет отвечать на запросы.

    ```json
      "runElevated": true,
      "runAsSystem": true,
    ```

    Пример:

    ```json
      {
          "type": "PowerShell",
          "name": "installFsLogix",
          "runElevated": true,
          "runAsSystem": true,
          "scriptUri": "https://raw.githubusercontent.com/danielsollondon/azvmimagebuilder/master/solutions/14_Building_Images_WVD/0_installConfFsLogix.ps1"
    ```
- Закомментируйте свой код. журнал сборки AIB (Настройка. log) является чрезвычайно подробным. Если вы запишете сценарии с помощью команды Write-Host, они будут отправлены в журналы и упростить устранение неполадок.

    ```PowerShell
     write-host 'AIB Customization: Starting OS Optimizations script'
    ```

- Коды выхода — AIB требует, чтобы все скрипты возвращали код выхода 0, а любой ненулевой код выхода приведет к сбою настройки AIB и остановке сборки. При наличии сложных скриптов добавьте коды завершения инструментирования и выдачи, которые будут отображаться в настройках. log.

    ```PowerShell
     Write-Host "Exit code: " $LASTEXITCODE
    ```
- Тест. Протестируйте и протестируйте код на автономной виртуальной машине, убедитесь, что нет запросов пользователя, вы используете правильную привилегию и т. д.

- Сети — `Set-NetAdapterAdvancedProperty` . Это задается в скрипте оптимизации, но завершается с ошибкой сборки AIB, так как она отключает сеть. Он находится в расследовании.

## <a name="prerequisites"></a>Предварительные требования

Необходимо установить последнюю версию командлетов Azure PowerShell. Дополнительные сведения об установке см. [здесь](https://docs.microsoft.com/powershell/azure/overview) .

```PowerShell
# Register for Azure Image Builder Feature
Register-AzProviderFeature -FeatureName VirtualMachineTemplatePreview -ProviderNamespace Microsoft.VirtualMachineImages

Get-AzProviderFeature -FeatureName VirtualMachineTemplatePreview -ProviderNamespace Microsoft.VirtualMachineImages

# wait until RegistrationState is set to 'Registered'

# check you are registered for the providers, ensure RegistrationState is set to 'Registered'.
Get-AzResourceProvider -ProviderNamespace Microsoft.VirtualMachineImages
Get-AzResourceProvider -ProviderNamespace Microsoft.Storage 
Get-AzResourceProvider -ProviderNamespace Microsoft.Compute
Get-AzResourceProvider -ProviderNamespace Microsoft.KeyVault

# If they do not saw registered, run the commented out code below.

## Register-AzResourceProvider -ProviderNamespace Microsoft.VirtualMachineImages
## Register-AzResourceProvider -ProviderNamespace Microsoft.Storage
## Register-AzResourceProvider -ProviderNamespace Microsoft.Compute
## Register-AzResourceProvider -ProviderNamespace Microsoft.KeyVault
```

## <a name="set-up-environment-and-variables"></a>Настройка среды и переменных

```azurepowershell-interactive
# Step 1: Import module
Import-Module Az.Accounts

# Step 2: get existing context
$currentAzContext = Get-AzContext

# destination image resource group
$imageResourceGroup="wvdImageDemoRg"

# location (see possible locations in main docs)
$location="westus2"

# your subscription, this will get your current subscription
$subscriptionID=$currentAzContext.Subscription.Id

# image template name
$imageTemplateName="wvd10ImageTemplate01"

# distribution properties object name (runOutput), i.e. this gives you the properties of the managed image on completion
$runOutputName="sigOutput"

# create resource group
New-AzResourceGroup -Name $imageResourceGroup -Location $location
```

## <a name="permissions-user-identity-and-role"></a>Разрешения, удостоверение пользователя и роль 


 Создайте удостоверение пользователя.

```azurepowershell-interactive
# setup role def names, these need to be unique
$timeInt=$(get-date -UFormat "%s")
$imageRoleDefName="Azure Image Builder Image Def"+$timeInt
$idenityName="aibIdentity"+$timeInt

## Add AZ PS modules to support AzUserAssignedIdentity and Az AIB
'Az.ImageBuilder', 'Az.ManagedServiceIdentity' | ForEach-Object {Install-Module -Name $_ -AllowPrerelease}

# create identity
New-AzUserAssignedIdentity -ResourceGroupName $imageResourceGroup -Name $idenityName

$idenityNameResourceId=$(Get-AzUserAssignedIdentity -ResourceGroupName $imageResourceGroup -Name $idenityName).Id
$idenityNamePrincipalId=$(Get-AzUserAssignedIdentity -ResourceGroupName $imageResourceGroup -Name $idenityName).PrincipalId

```

Назначьте разрешения удостоверению для распространения изображений. Эта команда скачивает и обновляет шаблон с указанными ранее параметрами.

```azurepowershell-interactive
$aibRoleImageCreationUrl="https://raw.githubusercontent.com/danielsollondon/azvmimagebuilder/master/solutions/12_Creating_AIB_Security_Roles/aibRoleImageCreation.json"
$aibRoleImageCreationPath = "aibRoleImageCreation.json"

# download config
Invoke-WebRequest -Uri $aibRoleImageCreationUrl -OutFile $aibRoleImageCreationPath -UseBasicParsing

((Get-Content -path $aibRoleImageCreationPath -Raw) -replace '<subscriptionID>',$subscriptionID) | Set-Content -Path $aibRoleImageCreationPath
((Get-Content -path $aibRoleImageCreationPath -Raw) -replace '<rgName>', $imageResourceGroup) | Set-Content -Path $aibRoleImageCreationPath
((Get-Content -path $aibRoleImageCreationPath -Raw) -replace 'Azure Image Builder Service Image Creation Role', $imageRoleDefName) | Set-Content -Path $aibRoleImageCreationPath

# create role definition
New-AzRoleDefinition -InputFile  ./aibRoleImageCreation.json

# grant role definition to image builder service principal
New-AzRoleAssignment -ObjectId $idenityNamePrincipalId -RoleDefinitionName $imageRoleDefName -Scope "/subscriptions/$subscriptionID/resourceGroups/$imageResourceGroup"
```

> [!NOTE] 
> Если отображается эта ошибка: "New-Азроледефинитион: превышен предел определения роли. Невозможно создать дополнительные определения ролей. чтобы устранить эту проблему, ознакомьтесь с этой статьей: https://docs.microsoft.com/azure/role-based-access-control/troubleshooting .



## <a name="create-the-shared-image-gallery"></a>Создание Общей коллекции образов 

Если у вас еще нет коллекции общих образов, ее необходимо создать.

```azurepowershell-interactive
$sigGalleryName= "myaibsig01"
$imageDefName ="win10wvd"

# create gallery
New-AzGallery -GalleryName $sigGalleryName -ResourceGroupName $imageResourceGroup  -Location $location

# create gallery definition
New-AzGalleryImageDefinition -GalleryName $sigGalleryName -ResourceGroupName $imageResourceGroup -Location $location -Name $imageDefName -OsState generalized -OsType Windows -Publisher 'myCo' -Offer 'Windows' -Sku '10wvd'

```

## <a name="configure-the-image-template"></a>Настройка шаблона образа

В этом примере у нас есть шаблон, готовый для загрузки и обновления шаблона с указанными ранее параметрами, он установит Фслогикс, оптимизации ОС, Microsoft Teams и запустит Центр обновления Windows в конце.

Если открыть шаблон, который вы видите в свойстве Source, то используемое изображение в этом примере использует многосеансовый образ Win 10. 

### <a name="windows-10-images"></a>Образы Windows 10
Существует два типа ключей, о которых следует знать: многосеансовый и один сеанс.

Многосеансовые образы предназначены для использования в составе пула. Ниже приведен пример сведений об образе в Azure.

```json
"publisher": "MicrosoftWindowsDesktop",
"offer": "Windows-10",
"sku": "20h2-evd",
"version": "latest"
```

Образы одного сеанса предназначены для индивидуального использования. Ниже приведен пример сведений об образе в Azure.

```json
"publisher": "MicrosoftWindowsDesktop",
"offer": "Windows-10",
"sku": "19h2-ent",
"version": "latest"
```

Вы также можете изменить доступные образы Win10:

```azurepowershell-interactive
Get-AzVMImageSku -Location westus2 -PublisherName MicrosoftWindowsDesktop -Offer windows-10
```

## <a name="download-template-and-configure"></a>Скачать шаблон и настроить

Теперь необходимо скачать шаблон и настроить его для использования.

```azurepowershell-interactive
$templateUrl="https://raw.githubusercontent.com/danielsollondon/azvmimagebuilder/master/solutions/14_Building_Images_WVD/armTemplateWVD.json"
$templateFilePath = "armTemplateWVD.json"

Invoke-WebRequest -Uri $templateUrl -OutFile $templateFilePath -UseBasicParsing

((Get-Content -path $templateFilePath -Raw) -replace '<subscriptionID>',$subscriptionID) | Set-Content -Path $templateFilePath
((Get-Content -path $templateFilePath -Raw) -replace '<rgName>',$imageResourceGroup) | Set-Content -Path $templateFilePath
((Get-Content -path $templateFilePath -Raw) -replace '<region>',$location) | Set-Content -Path $templateFilePath
((Get-Content -path $templateFilePath -Raw) -replace '<runOutputName>',$runOutputName) | Set-Content -Path $templateFilePath

((Get-Content -path $templateFilePath -Raw) -replace '<imageDefName>',$imageDefName) | Set-Content -Path $templateFilePath
((Get-Content -path $templateFilePath -Raw) -replace '<sharedImageGalName>',$sigGalleryName) | Set-Content -Path $templateFilePath
((Get-Content -path $templateFilePath -Raw) -replace '<region1>',$location) | Set-Content -Path $templateFilePath
((Get-Content -path $templateFilePath -Raw) -replace '<imgBuilderId>',$idenityNameResourceId) | Set-Content -Path $templateFilePath

```

Вы можете просмотреть [шаблон](https://raw.githubusercontent.com/danielsollondon/azvmimagebuilder/master/solutions/14_Building_Images_WVD/armTemplateWVD.json), и весь код доступен для просмотра.


## <a name="submit-the-template"></a>Отправка шаблона

Шаблон должен быть отправлен в службу, при этом будут скачаны все зависимые артефакты (например, сценарии), проверки, проверки разрешений и сохранены в промежуточной группе ресурсов с префиксом *IT_*.

```azurepowershell-interactive
New-AzResourceGroupDeployment -ResourceGroupName $imageResourceGroup -TemplateFile $templateFilePath -api-version "2020-02-14" -imageTemplateName $imageTemplateName -svclocation $location

# Optional - if you have any errors running the above, run:
$getStatus=$(Get-AzImageBuilderTemplate -ResourceGroupName $imageResourceGroup -Name $imageTemplateName)
$getStatus.ProvisioningErrorCode 
$getStatus.ProvisioningErrorMessage
```
 
## <a name="build-the-image"></a>Создание образа
```azurepowershell-interactive
Start-AzImageBuilderTemplate -ResourceGroupName $imageResourceGroup -Name $imageTemplateName -NoWait
```

> [!NOTE]
> Команда не будет ждать завершения сборки образа службой "Построитель образов". Вы можете запросить состояние ниже.

```azurepowershell-interactive
$getStatus=$(Get-AzImageBuilderTemplate -ResourceGroupName $imageResourceGroup -Name $imageTemplateName)

# this shows all the properties
$getStatus | Format-List -Property *

# these show the status the build
$getStatus.LastRunStatusRunState 
$getStatus.LastRunStatusMessage
$getStatus.LastRunStatusRunSubState
```
## <a name="create-a-vm"></a>Создание виртуальной машины
Теперь, когда сборка завершена, вы можете создать виртуальную машину из образа [, используя примеры отсюда.](https://docs.microsoft.com/powershell/module/az.compute/new-azvm#examples)

## <a name="clean-up"></a>Очистка

Сначала удалите шаблон группы ресурсов, не удаляя всю группу ресурсов, в противном случае промежуточная группа ресурсов (*IT_*), ИСПОЛЬЗУЕМая AIB, не будет очищена.

Удалите шаблон образа.

```azurepowershell-interactive
Remove-AzImageBuilderTemplate -ResourceGroupName $imageResourceGroup -Name wvd10ImageTemplate
```

Удаление назначения роли.

```azurepowershell-interactive
Remove-AzRoleAssignment -ObjectId $idenityNamePrincipalId -RoleDefinitionName $imageRoleDefName -Scope "/subscriptions/$subscriptionID/resourceGroups/$imageResourceGroup"

## remove definitions
Remove-AzRoleDefinition -Name "$idenityNamePrincipalId" -Force -Scope "/subscriptions/$subscriptionID/resourceGroups/$imageResourceGroup"

## delete identity
Remove-AzUserAssignedIdentity -ResourceGroupName $imageResourceGroup -Name $idenityName -Force
```

Удалите ее.

```azurepowershell-interactive
Remove-AzResourceGroup $imageResourceGroup -Force
```

## <a name="next-steps"></a>Дальнейшие действия

Вы можете попробовать дополнительные примеры [на сайте GitHub](https://github.com/danielsollondon/azvmimagebuilder/tree/master/quickquickstarts).
