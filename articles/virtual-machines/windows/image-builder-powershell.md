---
title: Создание виртуальной машины Windows с помощью Azure Image Builder с использованием PowerShell
description: Создайте виртуальную машину Windows с помощью модуля PowerShell для Azure Image Builder.
author: cynthn
ms.author: cynthn
ms.date: 03/02/2021
ms.topic: how-to
ms.service: virtual-machines
ms.subervice: image-builder
ms.colletion: windows
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 90d09763f2c9e167d6a0a34adbbc444ebad14c46
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101693464"
---
# <a name="preview-create-a-windows-vm-with-azure-image-builder-using-powershell"></a>Предварительная версия: создание виртуальной машины Windows с помощью Azure Image Builder с использованием PowerShell

В этой статье показано, как создать настраиваемый образ Windows с помощью модуля PowerShell для сборщика образов виртуальных машин Azure.

> [!CAUTION]
> Конструктор образов Azure сейчас поддерживается в общедоступной предварительной версии. Эта предварительная версия предоставляется без соглашения об уровне обслуживания. Эта версия не рекомендуется для использования с рабочими нагрузками в производственной среде. Некоторые функции могут не поддерживаться или их возможности могут быть ограничены. Дополнительные сведения см. в статье [Дополнительные условия использования предварительных выпусков Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Предварительные требования

Если у вас еще нет подписки Azure, создайте [бесплатную](https://azure.microsoft.com/free/) учетную запись Azure, прежде чем начинать работу.

Если вы решили использовать PowerShell локально, для работы с этой статьей установите модуль PowerShell Az и подключитесь к учетной записи Azure с помощью командлета [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount). См. сведения об [установке модуля Azure PowerShell](/powershell/azure/install-az-ps).

> [!IMPORTANT]
> Хотя модули PowerShell **AZ. имажебуилдер** и **AZ. манажедсервицеидентити** доступны в предварительной версии, их необходимо установить отдельно с помощью `Install-Module` командлета с `AllowPrerelease` параметром. После того как эти модули PowerShell станут общедоступными, они становятся частью будущих выпусков AZ PowerShell и доступны в рамках Azure Cloud Shell.

```azurepowershell-interactive
'Az.ImageBuilder', 'Az.ManagedServiceIdentity' | ForEach-Object {Install-Module -Name $_ -AllowPrerelease}
```

[!INCLUDE [cloud-shell-try-it](../../../includes/cloud-shell-try-it.md)]

Если вы используете несколько подписок Azure, выберите ту, за ресурсы в которой будут выставляться счета. Выберите требуемую подписку с помощью командлета [Set-AzContext](/powershell/module/az.accounts/set-azcontext).

```azurepowershell-interactive
Set-AzContext -SubscriptionId 00000000-0000-0000-0000-000000000000
```

### <a name="register-features"></a>Регистрация функций

Если вы впервые используете Azure Image Builder во время предварительной версии, зарегистрируйте новую функцию **виртуалмачинетемплатепревиев** .

```azurepowershell-interactive
Register-AzProviderFeature -ProviderNamespace Microsoft.VirtualMachineImages -FeatureName VirtualMachineTemplatePreview
```

Проверьте состояние регистрации функции.

> [!NOTE]
> **RegistrationState** может находиться в состоянии в `Registering` течение нескольких минут до перехода на `Registered` . Прежде чем продолжить, подождите, пока состояние не будет **зарегистрировано** .

```azurepowershell-interactive
Get-AzProviderFeature -ProviderNamespace Microsoft.VirtualMachineImages -FeatureName VirtualMachineTemplatePreview
```

Зарегистрируйте следующие поставщики ресурсов для использования с подпиской Azure, если они еще не зарегистрированы.

- Microsoft.Compute;
- Microsoft.KeyVault
- Microsoft.Storage;
- Microsoft.Network.
- Microsoft.VirtualMachineImages

```azurepowershell-interactive
Get-AzResourceProvider -ProviderNamespace Microsoft.Compute, Microsoft.KeyVault, Microsoft.Storage, Microsoft.VirtualMachineImages, Microsoft.Network |
  Where-Object RegistrationState -ne Registered |
    Register-AzResourceProvider
```

## <a name="define-variables"></a>Определение переменных

Некоторый фрагменты данных будут многократно использоваться. Создайте переменные для хранения информации.

```azurepowershell-interactive
# Destination image resource group name
$imageResourceGroup = 'myWinImgBuilderRG'

# Azure region
$location = 'WestUS2'

# Name of the image to be created
$imageTemplateName = 'myWinImage'

# Distribution properties of the managed image upon completion
$runOutputName = 'myDistResults'
```

Создайте переменную для идентификатора подписки Azure. Чтобы убедиться, что `subscriptionID` переменная содержит идентификатор подписки, можно выполнить вторую строку в следующем примере.

```azurepowershell-interactive
# Your Azure Subscription ID
$subscriptionID = (Get-AzContext).Subscription.Id
Write-Output $subscriptionID
```

## <a name="create-a-resource-group"></a>Создание группы ресурсов

Создайте [группу ресурсов Azure](../../azure-resource-manager/management/overview.md) с помощью командлета [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). Группа ресурсов — это логический контейнер, в котором ресурсы Azure развертываются и администрируются как группа.

В следующем примере создается группа ресурсов на основе имени в переменной `$imageResourceGroup` в регионе, указанном в переменной `$location`. Эта группа ресурсов используется для хранения артефакта шаблона конфигурации образа и образа.

```azurepowershell-interactive
New-AzResourceGroup -Name $imageResourceGroup -Location $location
```

## <a name="create-user-identity-and-set-role-permissions"></a>Создание удостоверений пользователей и задание разрешений роли

Предоставьте разрешения Azure Image Builder для создания образов в указанной группе ресурсов, используя следующий пример. Без этого разрешения процесс сборки образа не будет выполнен успешно.

Создайте переменные для определения роли и имен удостоверений. Эти значения должны быть однозначными.

```azurepowershell-interactive
[int]$timeInt = $(Get-Date -UFormat '%s')
$imageRoleDefName = "Azure Image Builder Image Def $timeInt"
$identityName = "myIdentity$timeInt"
```

Создайте удостоверение пользователя.

```azurepowershell-interactive
New-AzUserAssignedIdentity -ResourceGroupName $imageResourceGroup -Name $identityName
```

Сохраните идентификаторы ресурса удостоверения и субъекта в переменных.

```azurepowershell-interactive
$identityNameResourceId = (Get-AzUserAssignedIdentity -ResourceGroupName $imageResourceGroup -Name $identityName).Id
$identityNamePrincipalId = (Get-AzUserAssignedIdentity -ResourceGroupName $imageResourceGroup -Name $identityName).PrincipalId
```

### <a name="assign-permissions-for-identity-to-distribute-images"></a>Назначение разрешений удостоверению для распространения образов

Скачайте файл конфигурации. JSON и измените его в соответствии с параметрами, определенными в этой статье.

```azurepowershell-interactive
$myRoleImageCreationUrl = 'https://raw.githubusercontent.com/azure/azvmimagebuilder/master/solutions/12_Creating_AIB_Security_Roles/aibRoleImageCreation.json'
$myRoleImageCreationPath = "$env:TEMP\myRoleImageCreation.json"

Invoke-WebRequest -Uri $myRoleImageCreationUrl -OutFile $myRoleImageCreationPath -UseBasicParsing

$Content = Get-Content -Path $myRoleImageCreationPath -Raw
$Content = $Content -replace '<subscriptionID>', $subscriptionID
$Content = $Content -replace '<rgName>', $imageResourceGroup
$Content = $Content -replace 'Azure Image Builder Service Image Creation Role', $imageRoleDefName
$Content | Out-File -FilePath $myRoleImageCreationPath -Force
```

Создайте определение роли.

```azurepowershell-interactive
New-AzRoleDefinition -InputFile $myRoleImageCreationPath
```

Предоставьте определение роли для субъекта-службы Image Builder.

```azurepowershell-interactive
$RoleAssignParams = @{
  ObjectId = $identityNamePrincipalId
  RoleDefinitionName = $imageRoleDefName
  Scope = "/subscriptions/$subscriptionID/resourceGroups/$imageResourceGroup"
}
New-AzRoleAssignment @RoleAssignParams
```

> [!NOTE]
> Если появится сообщение об ошибке: "_New-азроледефинитион: превышен предел определения роли. Невозможно создать дополнительные определения ролей._ см. раздел [Устранение неполадок в Azure RBAC](../../role-based-access-control/troubleshooting.md).

## <a name="create-a-shared-image-gallery"></a>Создание Общей коллекции образов.

Создайте галерею.

```azurepowershell-interactive
$myGalleryName = 'myImageGallery'
$imageDefName = 'winSvrImages'

New-AzGallery -GalleryName $myGalleryName -ResourceGroupName $imageResourceGroup -Location $location
```

Создайте определение галереи.

```azurepowershell-interactive
$GalleryParams = @{
  GalleryName = $myGalleryName
  ResourceGroupName = $imageResourceGroup
  Location = $location
  Name = $imageDefName
  OsState = 'generalized'
  OsType = 'Windows'
  Publisher = 'myCo'
  Offer = 'Windows'
  Sku = 'Win2019'
}
New-AzGalleryImageDefinition @GalleryParams
```

## <a name="create-an-image"></a>Создание образа

Создайте исходный объект построителя образов Azure. Допустимые значения параметров см. [в статье Поиск образов виртуальных машин Windows в Azure Marketplace с помощью Azure PowerShell](./cli-ps-findimage.md) .

```azurepowershell-interactive
$SrcObjParams = @{
  SourceTypePlatformImage = $true
  Publisher = 'MicrosoftWindowsServer'
  Offer = 'WindowsServer'
  Sku = '2019-Datacenter'
  Version = 'latest'
}
$srcPlatform = New-AzImageBuilderSourceObject @SrcObjParams
```

Создайте объект распространителя Azure Image Builder.

```azurepowershell-interactive
$disObjParams = @{
  SharedImageDistributor = $true
  ArtifactTag = @{tag='dis-share'}
  GalleryImageId = "/subscriptions/$subscriptionID/resourceGroups/$imageResourceGroup/providers/Microsoft.Compute/galleries/$myGalleryName/images/$imageDefName"
  ReplicationRegion = $location
  RunOutputName = $runOutputName
  ExcludeFromLatest = $false
}
$disSharedImg = New-AzImageBuilderDistributorObject @disObjParams
```

Создайте объект настройки построителя образов Azure.

```azurepowershell-interactive
$ImgCustomParams01 = @{
  PowerShellCustomizer = $true
  CustomizerName = 'settingUpMgmtAgtPath'
  RunElevated = $false
  Inline = @("mkdir c:\\buildActions", "mkdir c:\\buildArtifacts", "echo Azure-Image-Builder-Was-Here  > c:\\buildActions\\buildActionsOutput.txt")
}
$Customizer01 = New-AzImageBuilderCustomizerObject @ImgCustomParams01
```

Создайте второй объект настройки построителя образов Azure.

```azurepowershell-interactive
$ImgCustomParams02 = @{
  FileCustomizer = $true
  CustomizerName = 'downloadBuildArtifacts'
  Destination = 'c:\\buildArtifacts\\index.html'
  SourceUri = 'https://raw.githubusercontent.com/azure/azvmimagebuilder/master/quickquickstarts/exampleArtifacts/buildArtifacts/index.html'
}
$Customizer02 = New-AzImageBuilderCustomizerObject @ImgCustomParams02
```

Создайте шаблон Azure Image Builder.

```azurepowershell-interactive
$ImgTemplateParams = @{
  ImageTemplateName = $imageTemplateName
  ResourceGroupName = $imageResourceGroup
  Source = $srcPlatform
  Distribute = $disSharedImg
  Customize = $Customizer01, $Customizer02
  Location = $location
  UserAssignedIdentityId = $identityNameResourceId
}
New-AzImageBuilderTemplate @ImgTemplateParams
```

По завершении возвращается сообщение и создается шаблон конфигурации построителя изображений в `$imageResourceGroup` .

Чтобы определить успешность процесса создания шаблона, можно использовать следующий пример.

```azurepowershell-interactive
Get-AzImageBuilderTemplate -ImageTemplateName $imageTemplateName -ResourceGroupName $imageResourceGroup |
  Select-Object -Property Name, LastRunStatusRunState, LastRunStatusMessage, ProvisioningState
```

В фоновом режиме построитель образов также создает промежуточную группу ресурсов в подписке. Эта группа ресурсов используется для сборки образа. Он имеет формат: `IT_<DestinationResourceGroup>_<TemplateName>` .

> [!WARNING]
> Не удаляйте промежуточную группу ресурсов напрямую. Удаление артефакта шаблона образа приведет к удалению промежуточной группы ресурсов.

Если служба сообщает о сбое во время отправки шаблона конфигурации образа:

- См. раздел [Устранение неполадок сборки образа виртуальной машины Azure (AIB)](../linux/image-builder-troubleshoot.md).
- Удалите шаблон, используя следующий пример, прежде чем повторять попытку.

```azurepowershell-interactive
Remove-AzImageBuilderTemplate -ImageTemplateName $imageTemplateName -ResourceGroupName $imageResourceGroup
```

## <a name="start-the-image-build"></a>Запуск сборки образа

Отправьте конфигурацию образа в службу "Построитель образов виртуальных машин".

```azurepowershell-interactive
Start-AzImageBuilderTemplate -ResourceGroupName $imageResourceGroup -Name $imageTemplateName
```

Дождитесь завершения процесса сборки образа. Этот шаг может занять до часа.

При возникновении ошибок ознакомьтесь с [разрешениями по устранению неполадок сборки образа виртуальной машины Azure (AIB)](../linux/image-builder-troubleshoot.md).

## <a name="create-a-vm"></a>Создание виртуальной машины

Сохраните в переменной учетные данные входа для виртуальной машины. Пароль должен быть сложным.

```azurepowershell-interactive
$Cred = Get-Credential
```

Создайте виртуальную машину с помощью созданного образа.

```azurepowershell-interactive
$ArtifactId = (Get-AzImageBuilderRunOutput -ImageTemplateName $imageTemplateName -ResourceGroupName $imageResourceGroup).ArtifactId

New-AzVM -ResourceGroupName $imageResourceGroup -Image $ArtifactId -Name myWinVM01 -Credential $Cred
```

## <a name="verify-the-customizations"></a>Проверка настроек

Подключитесь к виртуальной машине через подключение к удаленному рабочему столу, используя имя пользователя и пароль, заданные при создании виртуальной машины. В виртуальной машине откройте PowerShell и выполните команду `Get-Content` , как показано в следующем примере:

```azurepowershell-interactive
Get-Content -Path C:\buildActions\buildActionsOutput.txt
```

Вы должны увидеть выходные данные на основе содержимого файла, созданного в процессе настройки образа.

```Output
Azure-Image-Builder-Was-Here
```

В том же сеансе PowerShell убедитесь, что вторая настройка успешно завершена, проверив наличие файла, `c:\buildArtifacts\index.html` как показано в следующем примере:

```azurepowershell-interactive
Get-ChildItem c:\buildArtifacts\
```

Результатом должно быть список каталогов, в котором показан файл, скачанный в процессе настройки образа.

```Output
    Directory: C:\buildArtifacts

Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a---          29/01/2021    10:04            276 index.html
```


## <a name="clean-up-resources"></a>Очистка ресурсов

Если созданные при работе с этой статьей ресурсы не нужны, их можно удалить с помощью команды из следующего примера.

### <a name="delete-the-image-builder-template"></a>Удаление шаблона построителя образов

```azurepowershell-interactive
Remove-AzImageBuilderTemplate -ResourceGroupName $imageResourceGroup -Name $imageTemplateName
```

### <a name="delete-the-image-resource-group"></a>Удаление группы ресурсов образа

> [!CAUTION]
> Следующий пример удаляет указанную группу ресурсов и все содержащиеся в ней ресурсы.
> Если в указанной группе ресурсов существуют другие ресурсы, кроме созданных для этой статьи, они также будут удалены.

```azurepowershell-interactive
Remove-AzResourceGroup -Name $imageResourceGroup
```

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о компонентах JSON, использованных в этой статье, см. в разделе [Справочник по шаблонам для Image Builder](../linux/image-builder-json.md).
