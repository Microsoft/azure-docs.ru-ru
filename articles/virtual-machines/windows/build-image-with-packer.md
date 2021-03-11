---
title: Создание образов виртуальных машин с помощью пакета PowerShell
description: Узнайте, как использовать Pack и PowerShell для создания образов виртуальных машин в Azure.
author: cynthn
ms.service: virtual-machines
ms.subservice: imaging
ms.collection: windows
ms.topic: how-to
ms.workload: infrastructure
ms.date: 08/05/2020
ms.author: cynthn
ms.openlocfilehash: a2cd6152f9c8e75223fcf1aab77fee7700223979
ms.sourcegitcommit: 7edadd4bf8f354abca0b253b3af98836212edd93
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102561013"
---
# <a name="powershell-how-to-use-packer-to-create-virtual-machine-images-in-azure"></a>PowerShell: как использовать Pack для создания образов виртуальных машин в Azure
Каждая виртуальная машина в Azure создается из образа, который определяет дистрибутив Windows и версию операционной системы. Образы могут содержать предварительно установленные приложения и конфигурации. Azure Marketplace предоставляет большое количество образов Майкрософт и сторонних разработчиков для наиболее распространенных операционных систем и приложений. Кроме того, вы можете создать собственные настраиваемые образы, отвечающие конкретным потребностям. В этой статье описывается определение и создание пользовательских образов в Azure с использованием инструмента с открытым кодом [Packer](https://www.packer.io/).

Последняя проверка этой статьи выполнялась на 8/5/2020 с помощью средства [Pack](https://www.packer.io/docs/install) версии 1.6.1.

> [!NOTE]
> В Azure теперь есть служба "Конструктор образов Azure" (предварительная версия), которая используется для определения и создания собственных пользовательских образов. Так как Конструктор образов Azure создан на основе Packer, вы можете даже использовать имеющиеся скрипты подготовки оболочки Packer. Чтобы приступить к работе с Конструктором образов Azure, ознакомьтесь со статьей [Предварительный просмотр. Создание виртуальной машины Windows с помощью Конструктора образов Azure](image-builder.md).

## <a name="create-azure-resource-group"></a>Создание группы ресурсов Azure
В процессе сборки исходной виртуальной машины Packer создает временные ресурсы Azure. Чтобы сохранить эту исходную виртуальную машину для использования в качестве образа, необходимо определить группу ресурсов. Выходные данные процесса сборки Packer хранятся в этой группе ресурсов.

Создайте группу ресурсов с помощью командлета [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). В следующем примере создается группа ресурсов с именем *мипаккерграуп* в расположении *eastus* :

```azurepowershell
$rgName = "myPackerGroup"
$location = "East US"
New-AzResourceGroup -Name $rgName -Location $location
```

## <a name="create-azure-credentials"></a>Создание учетных данных Azure
Packer выполняет проверку подлинности с помощью субъекта-службы Azure. Субъект-служба Azure является удостоверением безопасности, которое можно использовать с приложениями, службами и средствами автоматизации, такими как Packer. Вы можете определять разрешения на то, какие операции может выполнять субъект-служба в Azure, и управлять ими.

Создайте субъект-службу с помощью [New азадсервицепринЦипал](/powershell/module/az.resources/new-azadserviceprincipal). Значение для `-DisplayName` должно быть уникальным. При необходимости замените его собственным значением.  

```azurepowershell
$sp = New-AzADServicePrincipal -DisplayName "PackerSP$(Get-Random)"
$BSTR = [System.Runtime.InteropServices.Marshal]::SecureStringToBSTR($sp.Secret)
$plainPassword = [System.Runtime.InteropServices.Marshal]::PtrToStringAuto($BSTR)
```

Затем выведите идентификатор приложения и пароль.

```powershell
$plainPassword
$sp.ApplicationId
```


Чтобы выполнить проверку подлинности в Azure, вам также необходимо получить идентификаторы вашего клиента и подписки Azure с помощью командлета [Get-AzSubscription](/powershell/module/az.accounts/get-azsubscription).

```powershell
Get-AzSubscription
```


## <a name="define-packer-template"></a>Определение шаблона Packer
Чтобы собрать образы, создайте шаблон в формате JSON. В шаблоне определите средства разработки и подготовки, выполняющие процесс сборки. Packer использует [средство разработки для Azure](https://www.packer.io/docs/builders/azure.html), которое позволяет определить ресурсы Azure, такие как учетные данные субъекта-службы, созданные на предыдущем шаге.

Создайте файл с именем *windows.json* и вставьте следующее содержимое. Введите свои значения следующим образом:

| Параметр                           | Где можно получить |
|-------------------------------------|----------------------------------------------------|
| *client_id*                         | Просмотрите идентификатор субъекта-службы с помощью `$sp.applicationId` |
| *client_secret*                     | Просмотрите автоматически созданный пароль с помощью `$plainPassword` |
| *tenant_id*                         | Выходные данные команды `$sub.TenantId` |
| *subscription_id*                   | Выходные данные команды `$sub.SubscriptionId` |
| *managed_image_resource_group_name* | Имя группы ресурсов, созданной на первом шаге |
| *managed_image_name*                | Имя создаваемого образа управляемого диска |

```json
{
  "builders": [{
    "type": "azure-arm",

    "client_id": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx",
    "client_secret": "ppppppp-pppp-pppp-pppp-ppppppppppp",
    "tenant_id": "zzzzzzz-zzzz-zzzz-zzzz-zzzzzzzzzzzz",
    "subscription_id": "yyyyyyy-yyyy-yyyy-yyyy-yyyyyyyyyyy",

    "managed_image_resource_group_name": "myPackerGroup",
    "managed_image_name": "myPackerImage",

    "os_type": "Windows",
    "image_publisher": "MicrosoftWindowsServer",
    "image_offer": "WindowsServer",
    "image_sku": "2016-Datacenter",

    "communicator": "winrm",
    "winrm_use_ssl": true,
    "winrm_insecure": true,
    "winrm_timeout": "5m",
    "winrm_username": "packer",

    "azure_tags": {
        "dept": "Engineering",
        "task": "Image deployment"
    },

    "build_resource_group_name": "myPackerGroup",
    "vm_size": "Standard_D2_v2"
  }],
  "provisioners": [{
    "type": "powershell",
    "inline": [
      "Add-WindowsFeature Web-Server",
      "while ((Get-Service RdAgent).Status -ne 'Running') { Start-Sleep -s 5 }",
      "while ((Get-Service WindowsAzureGuestAgent).Status -ne 'Running') { Start-Sleep -s 5 }",
      "& $env:SystemRoot\\System32\\Sysprep\\Sysprep.exe /oobe /generalize /quiet /quit",
      "while($true) { $imageState = Get-ItemProperty HKLM:\\SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\Setup\\State | Select ImageState; if($imageState.ImageState -ne 'IMAGE_STATE_GENERALIZE_RESEAL_TO_OOBE') { Write-Output $imageState.ImageState; Start-Sleep -s 10  } else { break } }"
    ]
  }]
}
```

Этот шаблон создает виртуальную машину Windows Server 2016, устанавливает службы IIS, а затем подготавливает виртуальную машину с помощью средства Sysprep. Установка IIS показывает, как можно использовать средство подготовки PowerShell для выполнения дополнительных команд. В этом случае окончательный образ Packer включает в себя также установочные файлы и параметры необходимого программного обеспечения.

Гостевой агент Windows участвует в процессе Sysprep. Чтобы виртуальную машину можно было сиспреп'ед, необходимо полностью установить агент. Чтобы убедиться, что это верно, перед выполнением sysprep.exe необходимо запустить все службы агента. В предыдущем фрагменте кода JSON показан один из способов сделать это в подокне подготовки PowerShell. Этот фрагмент необходим только в том случае, если виртуальная машина настроена для установки агента, что является значением по умолчанию.


## <a name="build-packer-image"></a>Создание образа Packer
Если средство Packer еще не установлено на локальном компьютере, [следуйте инструкциям по его установке](https://learn.hashicorp.com/packer/getting-started/install).

Создайте образ, открыв командную строку и указав файл шаблона Packer следующим образом:

```
./packer build windows.json
```

Ниже приведен пример выходных данных предыдущей команды.

```bash
azure-arm output will be in this color.

==> azure-arm: Running builder ...
    azure-arm: Creating Azure Resource Manager (ARM) client ...
==> azure-arm: Creating resource group ...
==> azure-arm:  -> ResourceGroupName : ‘packer-Resource-Group-pq0mthtbtt’
==> azure-arm:  -> Location          : ‘East US’
==> azure-arm:  -> Tags              :
==> azure-arm:  ->> task : Image deployment
==> azure-arm:  ->> dept : Engineering
==> azure-arm: Validating deployment template ...
==> azure-arm:  -> ResourceGroupName : ‘packer-Resource-Group-pq0mthtbtt’
==> azure-arm:  -> DeploymentName    : ‘pkrdppq0mthtbtt’
==> azure-arm: Deploying deployment template ...
==> azure-arm:  -> ResourceGroupName : ‘packer-Resource-Group-pq0mthtbtt’
==> azure-arm:  -> DeploymentName    : ‘pkrdppq0mthtbtt’
==> azure-arm: Getting the certificate’s URL ...
==> azure-arm:  -> Key Vault Name        : ‘pkrkvpq0mthtbtt’
==> azure-arm:  -> Key Vault Secret Name : ‘packerKeyVaultSecret’
==> azure-arm:  -> Certificate URL       : ‘https://pkrkvpq0mthtbtt.vault.azure.net/secrets/packerKeyVaultSecret/8c7bd823e4fa44e1abb747636128adbb'
==> azure-arm: Setting the certificate’s URL ...
==> azure-arm: Validating deployment template ...
==> azure-arm:  -> ResourceGroupName : ‘packer-Resource-Group-pq0mthtbtt’
==> azure-arm:  -> DeploymentName    : ‘pkrdppq0mthtbtt’
==> azure-arm: Deploying deployment template ...
==> azure-arm:  -> ResourceGroupName : ‘packer-Resource-Group-pq0mthtbtt’
==> azure-arm:  -> DeploymentName    : ‘pkrdppq0mthtbtt’
==> azure-arm: Getting the VM’s IP address ...
==> azure-arm:  -> ResourceGroupName   : ‘packer-Resource-Group-pq0mthtbtt’
==> azure-arm:  -> PublicIPAddressName : ‘packerPublicIP’
==> azure-arm:  -> NicName             : ‘packerNic’
==> azure-arm:  -> Network Connection  : ‘PublicEndpoint’
==> azure-arm:  -> IP Address          : ‘40.76.55.35’
==> azure-arm: Waiting for WinRM to become available...
==> azure-arm: Connected to WinRM!
==> azure-arm: Provisioning with Powershell...
==> azure-arm: Provisioning with shell script: /var/folders/h1/ymh5bdx15wgdn5hvgj1wc0zh0000gn/T/packer-powershell-provisioner902510110
    azure-arm: #< CLIXML
    azure-arm:
    azure-arm: Success Restart Needed Exit Code      Feature Result
    azure-arm: ------- -------------- ---------      --------------
    azure-arm: True    No             Success        {Common HTTP Features, Default Document, D...
    azure-arm: <Objs Version=“1.1.0.1” xmlns=“http://schemas.microsoft.com/powershell/2004/04"><Obj S=“progress” RefId=“0"><TN RefId=“0”><T>System.Management.Automation.PSCustomObject</T><T>System.Object</T></TN><MS><I64 N=“SourceId”>1</I64><PR N=“Record”><AV>Preparing modules for first use.</AV><AI>0</AI><Nil /><PI>-1</PI><PC>-1</PC><T>Completed</T><SR>-1</SR><SD> </SD></PR></MS></Obj></Objs>
==> azure-arm: Querying the machine’s properties ...
==> azure-arm:  -> ResourceGroupName : ‘packer-Resource-Group-pq0mthtbtt’
==> azure-arm:  -> ComputeName       : ‘pkrvmpq0mthtbtt’
==> azure-arm:  -> Managed OS Disk   : ‘/subscriptions/guid/resourceGroups/packer-Resource-Group-pq0mthtbtt/providers/Microsoft.Compute/disks/osdisk’
==> azure-arm: Powering off machine ...
==> azure-arm:  -> ResourceGroupName : ‘packer-Resource-Group-pq0mthtbtt’
==> azure-arm:  -> ComputeName       : ‘pkrvmpq0mthtbtt’
==> azure-arm: Capturing image ...
==> azure-arm:  -> Compute ResourceGroupName : ‘packer-Resource-Group-pq0mthtbtt’
==> azure-arm:  -> Compute Name              : ‘pkrvmpq0mthtbtt’
==> azure-arm:  -> Compute Location          : ‘East US’
==> azure-arm:  -> Image ResourceGroupName   : ‘myResourceGroup’
==> azure-arm:  -> Image Name                : ‘myPackerImage’
==> azure-arm:  -> Image Location            : ‘eastus’
==> azure-arm: Deleting resource group ...
==> azure-arm:  -> ResourceGroupName : ‘packer-Resource-Group-pq0mthtbtt’
==> azure-arm: Deleting the temporary OS disk ...
==> azure-arm:  -> OS Disk : skipping, managed disk was used...
Build ‘azure-arm’ finished.

==> Builds finished. The artifacts of successful builds are:
--> azure-arm: Azure.ResourceManagement.VMImage:

ManagedImageResourceGroupName: myResourceGroup
ManagedImageName: myPackerImage
ManagedImageLocation: eastus
```

Через несколько минут Packer создаст виртуальную машину, запустит средства подготовки и очистит развертывание.


## <a name="create-a-vm-from-the-packer-image"></a>Создание виртуальной машины на основе образа Packer
Теперь вы можете создать виртуальную машину из образа с помощью командлета [New-AzVM](/powershell/module/az.compute/new-azvm). Если они еще не существуют, создаются поддерживающие сетевые ресурсы. При появлении запроса введите имя пользователя с правами администратора и пароль. Этот пользователь будет создан на виртуальной машине. В следующем примере создается виртуальная машина *myVM* из образа *myPackerImage*.

```powershell
New-AzVm `
    -ResourceGroupName $rgName `
    -Name "myVM" `
    -Location $location `
    -VirtualNetworkName "myVnet" `
    -SubnetName "mySubnet" `
    -SecurityGroupName "myNetworkSecurityGroup" `
    -PublicIpAddressName "myPublicIpAddress" `
    -OpenPorts 80 `
    -Image "myPackerImage"
```

Если вы хотите создать виртуальные машины не в группе ресурсов или регионе, содержащем образ Packer, укажите идентификатор образа, а не его имя. Идентификатор образа можно получить, выполнив командлет [Get-AzImage](/powershell/module/az.compute/get-azimage).

Создание виртуальной машины из образа Packer занимает несколько минут.


## <a name="test-vm-and-webserver"></a>Тестирование виртуальной машины и веб-сервера
Получите общедоступный IP-адрес своей виртуальной машины с помощью командлета [Get-AzPublicIPAddress](/powershell/module/az.network/get-azpublicipaddress). Следующий пример позволяет получить IP-адрес для созданного ранее *myPublicIP*.

```powershell
Get-AzPublicIPAddress `
    -ResourceGroupName $rgName `
    -Name "myPublicIPAddress" | select "IpAddress"
```

Чтобы увидеть в действии виртуальную машину, которая включает в себя установленный экземпляр IIS из средства подготовки Packer, введите общедоступный IP-адрес в веб-браузере.

![Сайт IIS по умолчанию](./media/build-image-with-packer/iis.png) 


## <a name="next-steps"></a>Дальнейшие действия
С помощью [Конструктора образов Azure](image-builder.md) можно также использовать имеющиеся скрипты подготовки Packer.
