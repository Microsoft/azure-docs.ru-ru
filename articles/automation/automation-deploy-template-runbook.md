---
title: Развертывание шаблона Azure Resource Manager в runbook PowerShell службы автоматизации Azure
description: В этой статье описывается, как развернуть шаблон Azure Resource Manager, хранящийся в службе хранилища Azure, из модуля Runbook PowerShell.
services: automation
ms.subservice: process-automation
ms.date: 09/22/2020
ms.topic: conceptual
keywords: powershell, runbook, json, служба автоматизации azure
ms.openlocfilehash: d9b443d1840840d3d6202140da235589c73453cc
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99051402"
---
# <a name="deploy-an-azure-resource-manager-template-in-a-powershell-runbook"></a>Развертывание шаблона Azure Resource Manager в runbook PowerShell

Можно написать [runbook PowerShell службы автоматизации Azure](./learn/automation-tutorial-runbook-textual-powershell.md), который развертывает ресурс Azure с помощью [шаблона управления ресурсами Azure](../azure-resource-manager/templates/quickstart-create-templates-use-the-portal.md). Шаблоны позволяют использовать службу автоматизации Azure для автоматизации развертывания ресурсов Azure. Можно хранить шаблоны Resource Manager в центральном безопасном сетевом расположении, например в службе хранилища Azure.

В этой статье мы создадим runbook PowerShell, использующий шаблон Resource Manager, который расположен в [службе хранилища Azure](../storage/common/storage-introduction.md), для развертывания новой учетной записи хранения Azure.

## <a name="prerequisites"></a>Предварительные требования

* Подписка Azure. Если у вас ее нет, [активируйте преимущества для подписчиков MSDN](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/) или [зарегистрируйте бесплатную учетную запись](https://azure.microsoft.com/free/).
* [Учетная запись службы автоматизации](./automation-security-overview.md) , чтобы хранить модуль Runbook и выполнять проверку подлинности ресурсов Azure. Эта учетная запись должна иметь разрешение на запуск и остановку виртуальной машины.
* [Учетная запись хранения Azure](../storage/common/storage-account-create.md) , в которой будет храниться шаблон диспетчер ресурсов.
* Azure PowerShell, установленный на локальном компьютере. Дополнительные сведения о получении Azure PowerShell см. в статье [Установка модуля Azure PowerShell](/powershell/azure/install-az-ps).

## <a name="create-the-resource-manager-template"></a>Создание шаблона Resource Manager

В этом примере мы используем шаблон диспетчер ресурсов, который развертывает новую учетную запись хранения Azure.

Скопируйте приведенный ниже текст в текстовый редактор.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "storageAccountType": {
      "type": "string",
      "defaultValue": "Standard_LRS",
      "allowedValues": [
        "Standard_LRS",
        "Standard_GRS",
        "Standard_ZRS",
        "Premium_LRS"
      ],
      "metadata": {
        "description": "Storage Account type"
      }
    },
    "location": {
      "type": "string",
      "defaultValue": "[resourceGroup().location]",
      "metadata": {
        "description": "Location for all resources."
      }
    }
  },
  "variables": {
    "storageAccountName": "[concat(uniquestring(resourceGroup().id), 'standardsa')]"
  },
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "name": "[variables('storageAccountName')]",
      "apiVersion": "2018-02-01",
      "location": "[parameters('location')]",
      "sku": {
          "name": "[parameters('storageAccountType')]"
      },
      "kind": "Storage", 
      "properties": {
      }
    }
  ],
  "outputs": {
      "storageAccountName": {
          "type": "string",
          "value": "[variables('storageAccountName')]"
      }
  }
}
```

Сохраните файл локально как **TemplateTest.json**.

## <a name="save-the-resource-manager-template-in-azure-storage"></a>Сохранение шаблона Resource Manager в службе хранилища Azure

Теперь мы используем PowerShell, чтобы создать файловый ресурс службы хранилища Azure, и передадим в него файл **TemplateTest.json**. Инструкции по созданию файлового ресурса и передаче файла на портал Azure см. в разделе [Приступая к работе с хранилищем файлов Azure в Windows](../storage/files/storage-dotnet-how-to-use-files.md).

Запустите PowerShell на локальном компьютере и выполните приведенные команды, чтобы создать файловый ресурс и передать в него шаблон Resource Manager.

```powershell
# Log into Azure
Connect-AzAccount

# Get the access key for your storage account
$key = Get-AzStorageAccountKey -ResourceGroupName 'MyAzureAccount' -Name 'MyStorageAccount'

# Create an Azure Storage context using the first access key
$context = New-AzStorageContext -StorageAccountName 'MyStorageAccount' -StorageAccountKey $key[0].value

# Create a file share named 'resource-templates' in your Azure Storage account
$fileShare = New-AzStorageShare -Name 'resource-templates' -Context $context

# Add the TemplateTest.json file to the new file share
# "TemplatePath" is the path where you saved the TemplateTest.json file
$templateFile = 'C:\TemplatePath'
Set-AzStorageFileContent -ShareName $fileShare.Name -Context $context -Source $templateFile
```

## <a name="create-the-powershell-runbook-script"></a>Создание сценария runbook PowerShell

Теперь создадим сценарий PowerShell, который получает **TemplateTest.jsдля** файла из службы хранилища Azure и развертывает шаблон для создания новой учетной записи хранения Azure.

Вставьте приведенный ниже текст в текстовый редактор.

```powershell
param (
    [Parameter(Mandatory=$true)]
    [string]
    $ResourceGroupName,

    [Parameter(Mandatory=$true)]
    [string]
    $StorageAccountName,

    [Parameter(Mandatory=$true)]
    [string]
    $StorageAccountKey,

    [Parameter(Mandatory=$true)]
    [string]
    $StorageFileName
)

# Authenticate to Azure if running from Azure Automation
$ServicePrincipalConnection = Get-AutomationConnection -Name "AzureRunAsConnection"
Connect-AzAccount `
    -ServicePrincipal `
    -Tenant $ServicePrincipalConnection.TenantId `
    -ApplicationId $ServicePrincipalConnection.ApplicationId `
    -CertificateThumbprint $ServicePrincipalConnection.CertificateThumbprint | Write-Verbose

#Set the parameter values for the Resource Manager template
$Parameters = @{
    "storageAccountType"="Standard_LRS"
    }

# Create a new context
$Context = New-AzStorageContext -StorageAccountName $StorageAccountName -StorageAccountKey $StorageAccountKey

Get-AzStorageFileContent -ShareName 'resource-templates' -Context $Context -path 'TemplateTest.json' -Destination 'C:\Temp'

$TemplateFile = Join-Path -Path 'C:\Temp' -ChildPath $StorageFileName

# Deploy the storage account
New-AzResourceGroupDeployment -ResourceGroupName $ResourceGroupName -TemplateFile $TemplateFile -TemplateParameterObject $Parameters 
```

Сохраните файл локально как **DeployTemplate.ps1**.

## <a name="import-and-publish-the-runbook-into-your-azure-automation-account"></a>Импорт runbook в учетную запись службы автоматизации Azure и его публикация

Теперь мы используем PowerShell, чтобы импортировать runbook в учетную запись службы автоматизации Azure, а затем опубликовать его. Сведения о том, как импортировать и опубликовать runbook на портале Azure, см. в статье [Управление модулями runbook в службе автоматизации Azure](manage-runbooks.md).

Чтобы импортировать **DeployTemplate.ps1** в свою учетную запись автоматизации как runbook PowerShell, выполните следующие команды PowerShell.

```powershell
# MyPath is the path where you saved DeployTemplate.ps1
# MyResourceGroup is the name of the Azure ResourceGroup that contains your Azure Automation account
# MyAutomationAccount is the name of your Automation account
$importParams = @{
    Path = 'C:\MyPath\DeployTemplate.ps1'
    ResourceGroupName = 'MyResourceGroup'
    AutomationAccountName = 'MyAutomationAccount'
    Type = 'PowerShell'
}
Import-AzAutomationRunbook @importParams

# Publish the runbook
$publishParams = @{
    ResourceGroupName = 'MyResourceGroup'
    AutomationAccountName = 'MyAutomationAccount'
    Name = 'DeployTemplate'
}
Publish-AzAutomationRunbook @publishParams
```

## <a name="start-the-runbook"></a>Запуск модуля runbook

Теперь запустим runbook, вызвав командлет [Start-AzAutomationRunbook](/powershell/module/Az.Automation/Start-AzAutomationRunbook). Сведения о том, как запустить runbook на портале Azure, см. в разделе [Запуск модуля Runbook в службе автоматизации Azure](./start-runbooks.md).

В консоли PowerShell выполните следующие команды.

```powershell
# Set up the parameters for the runbook
$runbookParams = @{
    ResourceGroupName = 'MyResourceGroup'
    StorageAccountName = 'MyStorageAccount'
    StorageAccountKey = $key[0].Value # We got this key earlier
    StorageFileName = 'TemplateTest.json'
}

# Set up parameters for the Start-AzAutomationRunbook cmdlet
$startParams = @{
    ResourceGroupName = 'MyResourceGroup'
    AutomationAccountName = 'MyAutomationAccount'
    Name = 'DeployTemplate'
    Parameters = $runbookParams
}

# Start the runbook
$job = Start-AzAutomationRunbook @startParams
```

После запуска модуля Runbook можно проверить его состояние, извлекая значение свойства объекта Job `$job.Status` .

Модуль runbook получает шаблон Resource Manager и использует его для развертывания новой учетной записи хранения Azure. Чтобы увидеть, как создать учетную запись хранения, выполните следующую команду:

```powershell
Get-AzStorageAccount
```

## <a name="next-steps"></a>Дальнейшие действия

* Дополнительные сведения о шаблонах Resource Manager см. в статье [Общие сведения о диспетчере ресурсов Azure](../azure-resource-manager/management/overview.md).
* Чтобы приступить к работе со службой хранилища Azure, изучите раздел [Введение в хранилище Microsoft Azure](../storage/common/storage-introduction.md).
* Другие полезные runbook службы автоматизации Azure приведены в разделе [Использование runbook и других модулей службы автоматизации Azure](automation-runbook-gallery.md).
* Справочник по командлетам PowerShell см. в документации по [Az.Automation](/powershell/module/az.automation#automation).
