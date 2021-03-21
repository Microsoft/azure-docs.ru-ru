---
title: Использование расширений пользовательских скриптов для виртуальных машин на устройстве Azure Stack ребра Pro
description: Описание установки расширений пользовательских скриптов на виртуальных машинах, работающих на Azure Stack пограничных устройствах Pro с помощью шаблонов.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 02/22/2021
ms.author: alkohli
ms.openlocfilehash: 2d2e7d403ab3e9cc7e8e17de53b6e821ec24caa1
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102438018"
---
# <a name="deploy-custom-script-extension-on-vms-running-on-your-azure-stack-edge-pro-device"></a>Развертывание расширения настраиваемых скриптов на виртуальных машинах, работающих на устройстве Azure Stack ребра Pro

[!INCLUDE [applies-to-GPU-and-pro-r-and-mini-r-skus](../../includes/azure-stack-edge-applies-to-gpu-pro-r-mini-r-sku.md)]

Расширение пользовательских сценариев скачивает и запускает сценарии или команды на виртуальных машинах, запущенных на устройствах Azure Stack пограничных Pro. В этой статье подробно описано, как установить и запустить расширение пользовательских скриптов с помощью шаблона Azure Resource Manager. 

Эта статья относится к Azure Stackным графическим процессорам версии Pro, Azure Stackм пограничных Pro R и Azure Stack пограничных устройств R.

## <a name="about-custom-script-extension"></a>О расширении пользовательских скриптов

Расширение пользовательских скриптов полезно для настройки после развертывания, установки программного обеспечения или любой другой задачи настройки и управления. Вы можете скачать скрипты из службы хранилища Azure или другого доступного расположения в Интернете или предоставить скрипты или команды для среды выполнения расширения.

Расширение пользовательских сценариев интегрируется с шаблонами Azure Resource Manager. Его также можно запустить с помощью Azure CLI, PowerShell или REST API Виртуальных машин Azure.

## <a name="os-for-custom-script-extension"></a>ОС для расширения пользовательских сценариев

#### <a name="supported-os-for-custom-script-extension-on-windows"></a>Поддерживаемая ОС для расширения пользовательских сценариев в Windows

Расширение пользовательских скриптов для Windows будет выполняться в следующих ОС. Другие версии могут работать, но не тестировались на виртуальных машинах, работающих на Azure Stack пограничных устройствах Pro.

| Distribution | Версия |
|---|---|
| Windows Server 2019 | Основные сведения |
| Windows Server 2016 | Основные сведения |

#### <a name="supported-os-for-custom-script-extension-on-linux"></a>Поддерживаемая ОС для расширения пользовательских сценариев в Linux

Расширение пользовательских скриптов для Linux будет выполняться в следующих ОС. Другие версии могут работать, но не тестировались на виртуальных машинах, работающих на Azure Stack пограничных устройствах Pro.

| Distribution | Версия |
|---|---|
| Linux: Ubuntu | 18.04 LTS |
| Linux: Red Hat Enterprise Linux | 7,4, 7,5, 7,7 |

<!--### Script location

Instead of the scripts, in this article, we pass a command to execute via the Custom Script Extension. 

### Internet Connectivity

To download a script externally such as from GitHub or Azure Storage, make sure that the port on which you enable compute network, is connected to the internet. 

If your script is on a local server, then you may still need additional firewall and Network Security Group ports need to be opened.

> [!NOTE]
> Before you install the Custom Script extension, make sure that the port enabled for compute network on your device is connected to Internet and has access. -->

## <a name="prerequisites"></a>Предварительные условия

1. [Скачайте шаблоны виртуальных машин и файлы параметров](https://aka.ms/ase-vm-templates) на клиентский компьютер. Распакуйте загружаемый файл в каталог, который будет использоваться в качестве рабочего каталога.

1. На устройстве должна быть создана и развернута виртуальная машина. Чтобы создать виртуальные машины, выполните все действия, описанные в статье [развертывание виртуальной машины на Azure Stack пограничных Pro с помощью шаблонов](azure-stack-edge-gpu-deploy-virtual-machine-templates.md).

    Если вам нужно скачать сценарий, например из GitHub или службы хранилища Azure извне, во время настройки вычислений в сети включите порт, подключенный к Интернету для вычислений. Это позволяет скачать скрипт.

    В следующем примере порт 2 был подключен к Интернету и использовался для включения вычислений в сети. Если вы определили, что Kubernetes не требуется на предыдущем шаге, можно пропустить назначение IP-адреса и внешней службы для узла Kubernetes.

    ![Включить параметры вычислений для порта, подключенного к Интернету](media/azure-stack-edge-gpu-deploy-gpu-virtual-machine/enable-compute-network-1.png)

## <a name="install-custom-script-extension"></a>Установить расширение пользовательских скриптов

В зависимости от операционной системы для виртуальной машины можно установить расширение пользовательских скриптов для Windows или Linux.
 

### <a name="custom-script-extension-for-windows"></a>Расширение Custom Script в ОС Windows

Чтобы развернуть расширение настраиваемых скриптов для Windows для виртуальной машины, работающей на устройстве, измените `addCSExtWindowsVM.parameters.json` файл параметров, а затем разверните шаблон `addCSextensiontoVM.json` .

#### <a name="edit-parameters-file"></a>Изменение файла параметров

Файл `addCSExtWindowsVM.parameters.json` принимает следующие параметры:

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "vmName": {
            "value": "<Name of VM>" 
        },
        "extensionName": {
            "value": "<Name of extension>" 
        },
        "publisher": {
            "value": "Microsoft.Compute" 
        },
        "type": {
            "value": "CustomScriptExtension" 
        },
        "typeHandlerVersion": {
            "value": "1.10" 
        },
        "settings": {
            "value": {
                "commandToExecute" : "<Command to execute>"
            }
        }
    }
}
```
Укажите имя виртуальной машины, имя расширения и команду, которую требуется выполнить.

Ниже приведен пример файла параметров, который использовался в этой статье.

```powershell
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "vmName": {
            "value": "VM5" 
        },
        "extensionName": {
            "value": "CustomScriptExtension" 
        },
        "publisher": {
            "value": "Microsoft.Compute" 
        },
        "type": {
            "value": "CustomScriptExtension" 
        },
        "typeHandlerVersion": {
            "value": "1.10" 
        },
        "settings": {
            "value": {
                "commandToExecute" : "md C:\\Users\\Public\\Documents\\test"
            }
        }
    }
}
```
#### <a name="deploy-template"></a>Развертывание шаблона

Разверните шаблон `addCSextensiontoVM.json` . Этот шаблон развертывает расширение на существующей виртуальной машине. Выполните следующую команду.

```powershell
$templateFile = "<Path to addCSExtensiontoVM.json file>"
$templateParameterFile = "<Path to addCSExtWindowsVM.parameters.json file>"
$RGName = "<Resource group name>"
New-AzureRmResourceGroupDeployment -ResourceGroupName $RGName -TemplateFile $templateFile -TemplateParameterFile $templateParameterFile -Name "<Deployment name>"
```
> [!NOTE]
> Развертывание расширения — это длительное задание, выполнение которого занимает около 10 минут.

Ниже приведен пример выходных данных:

```powershell
PS C:\WINDOWS\system32> $templateFile = "C:\12-09-2020\ExtensionTemplates\addCSExtensiontoVM.json"
PS C:\WINDOWS\system32> $templateParameterFile = "C:\12-09-2020\ExtensionTemplates\addCSExtWindowsVM.parameters.json"
PS C:\WINDOWS\system32> $RGName = "myasegpuvm1"
PS C:\WINDOWS\system32> New-AzureRmResourceGroupDeployment -ResourceGroupName $RGName -TemplateFile $templateFile -TemplateParameterFile $templateParameterFile -Name "deployment7"

DeploymentName          : deployment7
ResourceGroupName       : myasegpuvm1
ProvisioningState       : Succeeded
Timestamp               : 12/17/2020 10:07:44 PM
Mode                    : Incremental
TemplateLink            :
Parameters              :
                          Name             Type                       Value
                          ===============  =========================  ==========
                          vmName           String                     VM5
                          extensionName    String                     CustomScriptExtension
                          publisher        String                     Microsoft.Compute
                          type             String                     CustomScriptExtension
                          typeHandlerVersion  String                     1.10
                          settings         Object                     {
                            "commandToExecute": "md C:\\Users\\Public\\Documents\\test"
                          }

Outputs                 :
DeploymentDebugLogLevel :

PS C:\WINDOWS\system32>
```
#### <a name="track-deployment"></a>Мониторинг развертывания

Чтобы проверить состояние развертывания расширений для данной виртуальной машины, выполните следующую команду: 

```powershell
Get-AzureRmVMExtension -ResourceGroupName <Name of resource group> -VMName <Name of VM> -Name <Name of the extension>
```
Ниже приведен пример выходных данных:

```powershell
PS C:\WINDOWS\system32> Get-AzureRmVMExtension -ResourceGroupName myasegpuvm1 -VMName VM5 -Name CustomScriptExtension

ResourceGroupName       : myasegpuvm1
VMName                  : VM5
Name                    : CustomScriptExtension
Location                : dbelocal
Etag                    : null
Publisher               : Microsoft.Compute
ExtensionType           : CustomScriptExtension
TypeHandlerVersion      : 1.10
Id                      : /subscriptions/947b3cfd-7a1b-4a90-7cc5-e52caf221332/resourceGroups/myasegpuvm1/providers/Microsoft.Compute/virtualMachines/VM5/extensions/CustomScriptExtension
PublicSettings          : {
                            "commandToExecute": "md C:\\Users\\Public\\Documents\\test"
                          }
ProtectedSettings       :
ProvisioningState       : Creating
Statuses                :
SubStatuses             :
AutoUpgradeMinorVersion : True
ForceUpdateTag          :

PS C:\WINDOWS\system32>
```

> [!NOTE]
> После завершения развертывания `ProvisioningState` изменения вносятся в `Succeeded` .

Выходные данные расширения регистрируются в файле, расположенном в следующей папке на целевой виртуальной машине. 

```cmd
C:\WindowsAzure\Logs\Plugins\Microsoft.Compute.CustomScriptExtension
```

Указанные файлы скачиваются в указанную ниже папку на целевой виртуальной машине.

```cmd
C:\Packages\Plugins\Microsoft.Compute.CustomScriptExtension\1.*\Downloads\<n>
```
<n> — десятичное целое число, которое может измениться между выполнениями расширения. Значение 1. * соответствует фактическому текущему `typeHandlerVersion` значению расширения. Например, фактическим каталогом в этом экземпляре было `C:\Packages\Plugins\Microsoft.Compute.CustomScriptExtension\1.10.9\Downloads\0` . 


В этом экземпляре команда для выполнения для пользовательского расширения: `md C:\\Users\\Public\\Documents\\test` . После успешной установки расширения можно проверить, что каталог был создан в виртуальной машине по указанному пути в команде. 


### <a name="custom-script-extension-for-linux"></a>Расширение Custom Script для Linux

Чтобы развернуть расширение настраиваемых скриптов для Windows для виртуальной машины, работающей на устройстве, измените `addCSExtLinuxVM.parameters.json` файл параметров, а затем разверните шаблон `addCSExtensiontoVM.json` .

#### <a name="edit-parameters-file"></a>Изменение файла параметров

Файл `addCSExtLinuxVM.parameters.json` принимает следующие параметры:

```powershell
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "vmName": {
            "value": "<Name of your VM>" 
        },
        "extensionName": {
            "value": "<Name of your extension>" 
        },
        "publisher": {
            "value": "Microsoft.Azure.Extensions" 
        },
        "type": {
            "value": "CustomScript" 
        },
        "typeHandlerVersion": {
            "value": "2.0" 
        },
        "settings": {
            "value": {
                "commandToExecute" : "<Command to execute>"
            }
        }
    }
}
```
Укажите имя виртуальной машины, имя расширения и команду, которую требуется выполнить.

Ниже приведен пример файла параметров, который использовался в этой статье:

```powershell
$templateFile = "<Path to addCSExtensionToVM.json file>"
$templateParameterFile = "<Path to addCSExtLinuxVM.parameters.json file>"
$RGName = "<Resource group name>"
New-AzureRmResourceGroupDeployment -ResourceGroupName $RGName -TemplateFile $templateFile -TemplateParameterFile $templateParameterFile -Name "<Deployment name>"
``` 

> [!NOTE]
> Развертывание расширения — это длительное задание, выполнение которого занимает около 10 минут.

Ниже приведен пример выходных данных:

```powershell
PS C:\WINDOWS\system32> $templateFile = "C:\12-09-2020\ExtensionTemplates\addCSExtensionToVM.json"
PS C:\WINDOWS\system32> $templateParameterFile = "C:\12-09-2020\ExtensionTemplates\addCSExtLinuxVM.parameters.json"
PS C:\WINDOWS\system32> $RGName = "myasegpuvm1"
PS C:\WINDOWS\system32> New-AzureRmResourceGroupDeployment -ResourceGroupName $RGName -TemplateFile $templateFile -TemplateParameterFile $templateParameterFile -Name "deployment99"

DeploymentName          : deployment99
ResourceGroupName       : myasegpuvm1
ProvisioningState       : Succeeded
Timestamp               : 12/18/2020 1:55:23 AM
Mode                    : Incremental
TemplateLink            :
Parameters              :
                          Name             Type                       Value
                          ===============  =========================  ==========
                          vmName           String                     VM6
                          extensionName    String                     LinuxCustomScriptExtension
                          publisher        String                     Microsoft.Azure.Extensions
                          type             String                     CustomScript
                          typeHandlerVersion  String                     2.0
                          settings         Object                     {
                            "commandToExecute": "sudo echo 'some text' >> /home/Administrator/file2.txt"
                          }

Outputs                 :
DeploymentDebugLogLevel :

PS C:\WINDOWS\system32>
```

`commandToExecute`Было задано создание файла `file2.txt` в `/home/Administrator` каталоге с содержимым файла `some text` . В этом случае можно проверить, что файл был создан по указанному пути.

```powershell
Administrator@VM6:~$ dir
file2.txt
Administrator@VM6:~$ cat file2.txt
some text
Administrator@VM6:
```

#### <a name="track-deployment-status"></a>Отслеживание состояния развертывания    
    
Шаблоны развертывания является долго выполняющимся заданием. Чтобы проверить состояние развертывания расширений для данной виртуальной машины, откройте другой сеанс PowerShell (Запуск от имени администратора). Выполните следующую команду. 

```powershell
Get-AzureRmVMExtension -ResourceGroupName myResourceGroup -VMName <VM Name> -Name <Extension Name>
```
Ниже приведен пример выходных данных: 

```powershell
PS C:\WINDOWS\system32> Get-AzureRmVMExtension -ResourceGroupName myasegpuvm1 -VMName VM5 -Name CustomScriptExtension

ResourceGroupName       : myasegpuvm1
VMName                  : VM5
Name                    : CustomScriptExtension
Location                : dbelocal
Etag                    : null
Publisher               : Microsoft.Compute
ExtensionType           : CustomScriptExtension
TypeHandlerVersion      : 1.10
Id                      : /subscriptions/947b3cfd-7a1b-4a90-7cc5-e52caf221332/resourceGroups/myasegpuvm1/providers/Microsoft.Compute/virtualMachines/VM5/extensions/CustomScriptExtension
PublicSettings          : {
                            "commandToExecute": "md C:\\Users\\Public\\Documents\\test"
                          }
ProtectedSettings       :
ProvisioningState       : Creating
Statuses                :
SubStatuses             :
AutoUpgradeMinorVersion : True
ForceUpdateTag          :

PS C:\WINDOWS\system32>
```

> [!NOTE]
> После завершения развертывания `ProvisioningState` изменения вносятся в `Succeeded` .

Выходные данные выполнения расширения записываются в следующий файл: `/var/lib/waagent/custom-script/download/0/` .


## <a name="remove-custom-script-extension"></a>Удалить расширение настраиваемых скриптов

Чтобы удалить расширение пользовательского скрипта, используйте следующую команду:

`Remove-AzureRmVMExtension -ResourceGroupName <Resource group name> -VMName <VM name> -Name <Extension name>`

Ниже приведен пример выходных данных:

```powershell
PS C:\WINDOWS\system32> Remove-AzureRmVMExtension -ResourceGroupName myasegpuvm1 -VMName VM6 -Name LinuxCustomScriptExtension
Virtual machine extension removal operation
This cmdlet will remove the specified virtual machine extension. Do you want to continue?
[Y] Yes  [N] No  [S] Suspend  [?] Help (default is "Y"): Yes
RequestId IsSuccessStatusCode StatusCode ReasonPhrase
--------- ------------------- ---------- ------------
                         True         OK OK
```


## <a name="next-steps"></a>Дальнейшие действия

[Командлеты Azure Resource Manager](/powershell/module/azurerm.resources/?view=azurermps-6.13.0&preserve-view=true)
