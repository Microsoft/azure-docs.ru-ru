---
title: Создание виртуальной машины в DevTest Labs с Azure PowerShell
description: Узнайте, как использовать Azure DevTest Labs для создания виртуальных машин и управления ими с помощью Azure PowerShell.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: f79b6ff92d633cf63477cddaabec918df352bec8
ms.sourcegitcommit: 15d27661c1c03bf84d3974a675c7bd11a0e086e6
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/09/2021
ms.locfileid: "102499260"
---
# <a name="create-a-virtual-machine-with-devtest-labs-using-azure-powershell"></a>Создание виртуальной машины с DevTest Labs с помощью Azure PowerShell
В этой статье показано, как создать виртуальную машину в Azure DevTest Labs с помощью Azure PowerShell. Сценарии PowerShell можно использовать для автоматизации создания виртуальных машин в лаборатории в Azure DevTest Labs. 

## <a name="prerequisites"></a>Предварительные требования
Перед началом работы

- [Создайте лабораторию](devtest-lab-create-lab.md) , если вы не хотите использовать существующую лабораторию для тестирования сценария или команд в этой статье. 
- [Установите Azure PowerShell](/powershell/azure/install-az-ps) или используйте Azure Cloud Shell, интегрированные в портал Azure. 

## <a name="powershell-script"></a>Сценарий PowerShell
В примере сценария в этом разделе используется командлет [Invoke-азресаурцеактион](/powershell/module/az.resources/invoke-azresourceaction) .  Этот командлет принимает идентификатор ресурса лаборатории, имя выполняемого действия ( `createEnvironment` ) и параметры, необходимые для выполнения этого действия. Параметры находятся в хэш-таблице, которая содержит все свойства описания виртуальной машины. 

```powershell
[CmdletBinding()]

Param(
[Parameter(Mandatory = $false)]  $SubscriptionId,
[Parameter(Mandatory = $true)]   $LabResourceGroup,
[Parameter(Mandatory = $true)]   $LabName,
[Parameter(Mandatory = $true)]   $NewVmName,
[Parameter(Mandatory = $true)]   $UserName,
[Parameter(Mandatory = $true)]   $Password
)
 
pushd $PSScriptRoot

try {
    if ($SubscriptionId -eq $null) {
        $SubscriptionId = (Get-AzContext).Subscription.SubscriptionId
    }

    $API_VERSION = '2016-05-15'
    $lab = Get-AzResource -ResourceId "/subscriptions/$SubscriptionId/resourceGroups/$LabResourceGroup/providers/Microsoft.DevTestLab/labs/$LabName"

    if ($lab -eq $null) {
       throw "Unable to find lab $LabName resource group $LabResourceGroup in subscription $SubscriptionId."
    }

    #For this example, we are getting the first allowed subnet in the first virtual network
    #  for the lab.
    #If a specific virtual network is needed use | to find it. 
    #ie $virtualNetwork = @(Get-AzResource -ResourceType  'Microsoft.DevTestLab/labs/virtualnetworks' -ResourceName $LabName -ResourceGroupName $lab.ResourceGroupName -ApiVersion $API_VERSION) | Where-Object Name -EQ "SpecificVNetName"

    $virtualNetwork = @(Get-AzResource -ResourceType  'Microsoft.DevTestLab/labs/virtualnetworks' -ResourceName $LabName -ResourceGroupName $lab.ResourceGroupName -ApiVersion $API_VERSION)[0]

    $labSubnetName = $virtualNetwork.properties.allowedSubnets[0].labSubnetName

    #Prepare all the properties needed for the createEnvironment
    # call used to create the new VM.
    # The properties will be slightly different depending on the base of the vm
    # (a marketplace image, custom image or formula).
    # The setup of the virtual network to be used may also affect the properties.
    # This sample includes the properties to add an additional disk under dataDiskParameters
    
    $parameters = @{
       "name"      = $NewVmName;
       "location"  = $lab.Location;
       "properties" = @{
          "labVirtualNetworkId"     = $virtualNetwork.ResourceId;
          "labSubnetName"           = $labSubnetName;
          "notes"                   = "Windows Server 2016 Datacenter";
          "osType"                  = "windows"
          "expirationDate"          = "2019-12-01"
          "galleryImageReference"   = @{
             "offer"     = "WindowsServer";
             "publisher" = "MicrosoftWindowsServer";
             "sku"       = "2016-Datacenter";
             "osType"    = "Windows";
             "version"   = "latest"
          };
          "size"                    = "Standard_DS2_v2";
          "userName"                = $UserName;
          "password"                = $Password;
          "disallowPublicIpAddress" = $true;
          "dataDiskParameters" = @(@{
            "attachNewDataDiskOptions" = @{
                "diskName" = "adddatadisk"
                "diskSizeGiB" = "1023"
                "diskType" = "Standard"
                }
          "hostCaching" = "ReadWrite"
          })
       }
    }
    
    #The following line is the same as invoking
    # https://azure.github.io/projects/apis/#!/Labs/Labs_CreateEnvironment rest api

    Invoke-AzResourceAction -ResourceId $lab.ResourceId -Action 'createEnvironment' -Parameters $parameters -ApiVersion $API_VERSION -Force -Verbose
}
finally {
   popd
}
```

Свойства виртуальной машины в приведенном выше сценарии позволяют создать виртуальную машину с Windows Server 2016 Datacenter в качестве ОС. Для каждого типа виртуальной машины эти свойства будут немного отличаться. В разделе [Определение виртуальной машины](#define-virtual-machine) показано, как определить, какие свойства следует использовать в этом скрипте.

Следующая команда предоставляет пример запуска сценария, сохраненного в имени файла: Create-LabVirtualMachine.ps1. 

```powershell
 PS> .\Create-LabVirtualMachine.ps1 -ResourceGroupName 'MyLabResourceGroup' -LabName 'MyLab' -userName 'AdminUser' -password 'Password1!' -VMName 'MyLabVM'
```

## <a name="define-virtual-machine"></a>Определение виртуальной машины
В этом разделе показано, как получить свойства, относящиеся к типу виртуальной машины, которую необходимо создать. 

### <a name="use-azure-portal"></a>Использование портала Azure
При создании виртуальной машины в портал Azure можно создать шаблон Azure Resource Manager. Вам не нужно завершать процесс создания виртуальной машины. Выполняйте эти действия только после того, как будет отображен шаблон. Это лучший способ получить необходимое описание JSON, если виртуальная машина лаборатории еще не создана. 

1. Перейдите на [портал Azure](https://portal.azure.com).
2. В меню навигации слева выберите **все службы** .
3. Найдите и выберите **DevTest Labs** из списка служб. 
4. На странице **DevTest Labs** (лабораторные занятия) выберите свою лабораторию в списке лабораторий.
5. На домашней странице лаборатории выберите **+ Добавить** на панели инструментов. 
6. Выберите **базовый образ** для виртуальной машины. 
7. Выберите **Параметры автоматизации** в нижней части страницы над кнопкой **Отправить** . 
8. Вы увидите **шаблон Azure Resource Manager** для создания виртуальной машины. 
9. Сегмент JSON в разделе **ресурсов** содержит определение выбранного ранее типа образа. 

    ```json
    {
      "apiVersion": "2018-10-15-preview",
      "type": "Microsoft.DevTestLab/labs/virtualmachines",
      "name": "[variables('vmName')]",
      "location": "[resourceGroup().location]",
      "properties": {
        "labVirtualNetworkId": "[variables('labVirtualNetworkId')]",
        "notes": "Windows Server 2019 Datacenter",
        "galleryImageReference": {
          "offer": "WindowsServer",
          "publisher": "MicrosoftWindowsServer",
          "sku": "2019-Datacenter",
          "osType": "Windows",
          "version": "latest"
        },
        "size": "[parameters('size')]",
        "userName": "[parameters('userName')]",
        "password": "[parameters('password')]",
        "isAuthenticationWithSshKey": false,
        "labSubnetName": "[variables('labSubnetName')]",
        "disallowPublicIpAddress": true,
        "storageType": "Standard",
        "allowClaim": false,
        "networkInterface": {
          "sharedPublicIpAddressConfiguration": {
            "inboundNatRules": [
              {
                "transportProtocol": "tcp",
                "backendPort": 3389
              }
            ]
          }
        }
      }
    }
    ```

В этом примере показано, как получить определение образа на рынке Azure. Таким же образом можно получить определение пользовательского образа, формулы или среды. Добавьте артефакты, необходимые для виртуальной машины, и задайте необходимые дополнительные параметры. После предоставления значений для обязательных полей и любых дополнительных полей перед нажатием кнопки **Параметры автоматизации** .

### <a name="use-azure-rest-api"></a>Использование Azure REST API
Следующая процедура позволяет получить свойства образа с помощью REST API. Эти действия работают только для существующей виртуальной машины в лаборатории. 

1. Перейдите на страницу " [виртуальные машины — список](/rest/api/dtl/virtualmachines/list) " и нажмите кнопку " **попробовать** ". 
2. Выберите свою **подписку Azure**.
3. Введите **группу ресурсов для лаборатории**.
4. Введите **имя лаборатории**. 
5. Выберите **Запуск**.
6. Вы увидите **свойства для образа** , на основе которого была создана виртуальная машина. 

## <a name="set-expiration-date"></a>Дата истечения срока действия
В таких сценариях, как обучение, демонстрации и пробные версии, вы можете создавать виртуальные машины и удалять их автоматически после фиксированной длительности, чтобы не тратить излишнее издержки. Вы можете задать дату истечения срока действия виртуальной машины при ее создании с помощью PowerShell, как показано в примере раздела [сценария PowerShell](#powershell-script) .

Ниже приведен пример скрипта PowerShell, который задает дату истечения срока действия для всех существующих виртуальных машин в лаборатории.

```powershell
# Values to change
$subscriptionId = '<Enter the subscription Id that contains lab>'
$labResourceGroup = '<Enter the lab resource group>'
$labName = '<Enter the lab name>'
$VmName = '<Enter the VmName>'
$expirationDate = '<Enter the expiration date e.g. 2019-12-16>'

# Log into your Azure account
Login-AzureRmAccount

Select-AzureRmSubscription -SubscriptionId $subscriptionId
$VmResourceId = "subscriptions/$subscriptionId/resourcegroups/$labResourceGroup/providers/microsoft.devtestlab/labs/$labName/virtualmachines/$VmName"

$vm = Get-AzureRmResource -ResourceId $VmResourceId -ExpandProperties

# Get all the Vm properties
$VmProperties = $vm.Properties

# Set the expirationDate property
If ($VmProperties.expirationDate -eq $null) {
    $VmProperties | Add-Member -MemberType NoteProperty -Name expirationDate -Value $expirationDate
} Else {
    $VmProperties.expirationDate = $expirationDate
}

Set-AzureRmResource -ResourceId $VmResourceId -Properties $VmProperties -Force
```


## <a name="next-steps"></a>Дальнейшие действия
См. следующее содержимое: [Azure PowerShell документация по Azure DevTest Labs](/powershell/module/az.devtestlabs/)
