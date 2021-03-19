---
title: Создание пользовательского образа из VHD-файла с помощью Azure PowerShell
description: Автоматизация создания пользовательского образа в Azure DevTest Labs из VHD-файла с помощью PowerShell
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: 4b0712fdbec1ce23ad9e09d972e425cb7941107b
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "87288984"
---
# <a name="create-a-custom-image-from-a-vhd-file-using-powershell"></a>Создание пользовательского образа из VHD-файла с помощью PowerShell

[!INCLUDE [devtest-lab-create-custom-image-from-vhd-selector](../../includes/devtest-lab-create-custom-image-from-vhd-selector.md)]

[!INCLUDE [devtest-lab-custom-image-definition](../../includes/devtest-lab-custom-image-definition.md)]

[!INCLUDE [devtest-lab-upload-vhd-options](../../includes/devtest-lab-upload-vhd-options.md)]

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="step-by-step-instructions"></a>Пошаговые инструкции

Чтобы создать пользовательский образ из VHD-файла с помощью PowerShell, сделайте следующее:

1. В командной строке PowerShell Войдите в свою учетную запись Azure, выполнив следующий вызов командлета **Connect-азаккаунт** .

    ```powershell
    Connect-AzAccount
    ```

1.  Выберите нужную подписку Azure, вызвав командлет **SELECT-азсубскриптион** . Замените заполнитель для переменной **$subscriptionId** на идентификатор действующей подписки Azure.

    ```powershell
    $subscriptionId = '<Specify your subscription ID here>'
    Select-AzSubscription -SubscriptionId $subscriptionId
    ```

1.  Получите лабораторный объект, вызвав командлет **Get-азресаурце** . Замените заполнители для переменных **$labRg** и **$labName** на соответствующие значения для своей среды.

    ```powershell
    $labRg = '<Specify your lab resource group name here>'
    $labName = '<Specify your lab name here>'
    $lab = Get-AzResource -ResourceId ('/subscriptions/' + $subscriptionId + '/resourceGroups/' + $labRg + '/providers/Microsoft.DevTestLab/labs/' + $labName)
    ```

1.  Замените заполнитель для переменной **$vhdUri** на URI своего VHD-файла. URI VHD-файла можно получить в колонке учетной записи хранилища BLOB-объектов на портале Azure.

    ```powershell
    $vhdUri = '<Specify the VHD URI here>'
    ```

1.  Создайте пользовательский образ с помощью командлета **New-азресаурцеграупдеплоймент** . Замените следующие заполнители для переменных **$customImageName** и **$customImageDescription** на значимые имена для своей среды.

    ```powershell
    $customImageName = '<Specify the custom image name>'
    $customImageDescription = '<Specify the custom image description>'

    $parameters = @{existingLabName="$($lab.Name)"; existingVhdUri=$vhdUri; imageOsType='windows'; isVhdSysPrepped=$false; imageName=$customImageName; imageDescription=$customImageDescription}

    New-AzResourceGroupDeployment -ResourceGroupName $lab.ResourceGroupName -Name CreateCustomImage -TemplateUri 'https://raw.githubusercontent.com/Azure/azure-devtestlab/master/samples/DevTestLabs/QuickStartTemplates/201-dtl-create-customimage-from-vhd/azuredeploy.json' -TemplateParameterObject $parameters
    ```

## <a name="powershell-script-to-create-a-custom-image-from-a-vhd-file"></a>Создание пользовательского образа из VHD-файла с помощью сценария PowerShell

Следующий сценарий PowerShell позволяет создать пользовательский образ из VHD-файла. Замените заполнители, выделенные угловыми скобками, на необходимые значения.

```powershell
# Log in to your Azure account.
Connect-AzAccount

# Select the desired Azure subscription.
$subscriptionId = '<Specify your subscription ID here>'
Select-AzSubscription -SubscriptionId $subscriptionId

# Get the lab object.
$labRg = '<Specify your lab resource group name here>'
$labName = '<Specify your lab name here>'
$lab = Get-AzResource -ResourceId ('/subscriptions/' + $subscriptionId + '/resourceGroups/' + $labRg + '/providers/Microsoft.DevTestLab/labs/' + $labName)

# Set the URI of the VHD file.
$vhdUri = '<Specify the VHD URI here>'

# Set the custom image name and description values.
$customImageName = '<Specify the custom image name>'
$customImageDescription = '<Specify the custom image description>'

# Set up the parameters object.
$parameters = @{existingLabName="$($lab.Name)"; existingVhdUri=$vhdUri; imageOsType='windows'; isVhdSysPrepped=$false; imageName=$customImageName; imageDescription=$customImageDescription}

# Create the custom image.
New-AzResourceGroupDeployment -ResourceGroupName $lab.ResourceGroupName -Name CreateCustomImage -TemplateUri 'https://raw.githubusercontent.com/Azure/azure-devtestlab/master/samples/DevTestLabs/QuickStartTemplates/201-dtl-create-customimage-from-vhd/azuredeploy.json' -TemplateParameterObject $parameters
```

## <a name="related-blog-posts"></a>Связанные записи в блогах

- [Custom images or formulas? (Пользовательские изображения или формулы?)](./devtest-lab-faq.md#blog-post)
- [Copying Custom Images between Azure DevTest Labs (Копирование пользовательских образов между лабораториями для разработки и тестирования Azure)](https://www.visualstudiogeeks.com/blog/DevOps/How-To-Move-CustomImages-VHD-Between-AzureDevTestLabs#copying-custom-images-between-azure-devtest-labs)

## <a name="next-steps"></a>Дальнейшие действия

- [Добавление виртуальной машины с артефактами в лабораторию в Azure DevTest Labs](devtest-lab-add-vm.md)
