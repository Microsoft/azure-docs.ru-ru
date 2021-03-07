---
title: Обзор и развертывание виртуальных машин GPU на устройстве Azure Stack ребра Pro
description: В этой статье описывается создание виртуальных машин GPU на Azure Stack пограничном устройстве Pro и управление ими с помощью шаблонов.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 02/22/2021
ms.author: alkohli
ms.openlocfilehash: ff805b758dce05a66764ab1ff08e53378c946362
ms.sourcegitcommit: 5bbc00673bd5b86b1ab2b7a31a4b4b066087e8ed
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/07/2021
ms.locfileid: "102438188"
---
# <a name="gpu-vms-for-your-azure-stack-edge-pro-device"></a>Виртуальные машины GPU для устройства Azure Stack ребра Pro

[!INCLUDE [applies-to-GPU-and-pro-r-skus](../../includes/azure-stack-edge-applies-to-gpu-pro-r-sku.md)]

В этой статье содержатся общие сведения о виртуальных машинах GPU на устройстве Azure Stack. В этой статье описывается, как создать виртуальную машину GPU, а затем установить расширение драйвера GPU для установки соответствующих драйверов NVIDIA. Используйте шаблоны Azure Resource Manager, чтобы создать виртуальную машину GPU и установить расширение драйвера GPU. 

Эта статья относится к Azure Stackным графическим процессорам версии Pro и Azure Stackным устройствам с пограничным Pro R.

## <a name="about-gpu-vms"></a>О виртуальных машинах GPU

Устройства Azure Stack пограничной Pro оснащены 1 или 2 из Tesla T4 GPU NVIDIA. Чтобы развернуть на этих устройствах рабочие нагрузки виртуальных машин с ускорением GPU, используйте размеры виртуальных машин, оптимизированные для GPU. Например, для развертывания рабочих нагрузок вывода, в которых используются T4 GPU, следует использовать серию NC серии v3. 

Дополнительные сведения см. в статье о [виртуальных машинах серии T4 версии 3](../virtual-machines/nct4-v3-series.md).

## <a name="supported-os-and-gpu-drivers"></a>Поддерживаемые драйверы ОС и GPU 

Чтобы воспользоваться преимуществами возможностей GPU виртуальных машин Azure серии N, необходимо установить драйверы GPU NVIDIA. 

Расширение драйвера GPU NVIDIA устанавливает соответствующие драйверы NVIDIA CUDA или GRID. Вы можете установить расширение или управлять им с помощью шаблонов Azure Resource Manager.

### <a name="supported-os-for-gpu-extension-for-windows"></a>Поддерживаемая ОС для расширения GPU для Windows

Это расширение поддерживает следующие операционные системы (OSs). Другие версии могут работать, но не проверялись на виртуальных машинах GPU, работающих на Azure Stack на устройствах с пограничным Pro.

| Distribution | Версия |
|---|---|
| Windows Server 2019 | Основные сведения |
| Windows Server 2016 | Основные сведения |

### <a name="supported-os-for-gpu-extension-for-linux"></a>Поддерживаемая ОС для расширения GPU для Linux

Это расширение поддерживает следующие дистрибутивов операционной системы в зависимости от поддержки драйвера для конкретной версии ОС. Другие версии могут работать, но не проверялись на виртуальных машинах GPU, работающих на Azure Stack на устройствах с пограничным Pro.


| Distribution | Версия |
|---|---|
| Ubuntu | 18.04 LTS |
| Red Hat Enterprise Linux | 7.4 |


## <a name="gpu-vms-and-kubernetes"></a>Виртуальные машины GPU и Kubernetes

Перед развертыванием виртуальных машин GPU на устройстве ознакомьтесь со следующими вопросами, если на устройстве настроен Kubernetes.

#### <a name="for-1-gpu-device"></a>Для устройства с 1-GPU: 

- **Создайте виртуальную машину GPU, а затем конфигурацию Kubernetes на устройстве**. в этом случае будет успешно выполнено создание ВИРТУАЛЬНОЙ машины GPU и конфигурация Kubernetes. В данном случае Kubernetes не будет иметь доступ к GPU.

- **Настройте Kubernetes на устройстве, а затем создание виртуальной машины GPU**. в этом сценарии Kubernetes ВЫДАСТ графический процессор на устройстве, и создание виртуальной машины завершится сбоем, так как ресурсы GPU недоступны.

#### <a name="for-2-gpu-device"></a>Для устройства с 2 GPU

- **Создайте виртуальную машину GPU, а затем конфигурацию Kubernetes на устройстве**. в этом случае виртуальная машина GPU, созданная вами, запросит один GPU на устройстве, а конфигурация Kubernetes будет успешной и запросит оставшийся один GPU. 

- **Создайте две виртуальные машины GPU, за которыми следует конфигурация Kubernetes на устройстве**. в этом случае две виртуальные машины GPU будут запрашивать два GPU на устройстве, а Kubernetes успешно настроены без GPU. 

- **Настройка Kubernetes на устройстве с последующим созданием виртуальной машины GPU**. в этом случае Kubernetes будет запрашивать как GPU на устройстве, так и создание виртуальной машины, так как ресурсы GPU недоступны.

Если на вашем устройстве работают виртуальные машины GPU, а также настроен Kubernetes, то при каждом освобождении ВИРТУАЛЬНОЙ машины (когда вы останавливаете или удаляете виртуальную машину, используя Stop-AzureRmVM или Remove-AzureRmVM), существует риск того, что кластер Kubernetes будет запрашивать все GPU, доступные на устройстве. В таком случае вы не сможете перезапустить виртуальные машины GPU, развернутые на устройстве, или создать виртуальные машины GPU.


## <a name="create-gpu-vms"></a>Создание виртуальных машин GPU

Выполните следующие действия при развертывании виртуальных машин GPU на устройстве:

1. Выясните, будет ли устройство работать под Kubernetes. Если устройство запустит Kubernetes, необходимо сначала создать виртуальную машину GPU, а затем настроить Kubernetes. Если сначала настроить Kubernetes, то будут запрашиваться все доступные ресурсы GPU, и создание виртуальной машины GPU завершится сбоем.

1. [Скачайте шаблоны виртуальных машин и файлы параметров](https://aka.ms/ase-vm-templates) на клиентский компьютер. Распакуйте его в каталог, который будет использоваться в качестве рабочего каталога.

1. Чтобы создать виртуальные машины GPU, выполните все действия, описанные в статье [развертывание виртуальной машины на Azure Stack пограничной Pro с помощью шаблонов](azure-stack-edge-gpu-deploy-virtual-machine-templates.md) , за исключением следующих отличий: 

    1. При настройке сети вычислений включите порт, подключенный к Интернету, для вычислений. Это позволяет скачать драйверы GPU, необходимые для расширений GPU для виртуальных машин GPU.

        Ниже приведен пример, в котором порт 2 был подключен к Интернету и был использован для включения вычислений в сети. Если вы определили, что на предыдущем шаге Kubernetes не требуется, можно пропустить назначение IP-адреса и внешней службы для узла Kubernetes.

        ![Включить параметры вычислений для порта, подключенного к Интернету](media/azure-stack-edge-gpu-deploy-gpu-virtual-machine/enable-compute-network-1.png)

            
    1. Создайте виртуальную машину с помощью шаблонов. При указании размеров виртуальных машин GPU обязательно используйте NCasT4-v3-Series в, `CreateVM.parameters.json` так как они поддерживаются для виртуальных машин GPU. Дополнительные сведения см. в статье [Поддерживаемые размеры виртуальных машин для виртуальных машин GPU](azure-stack-edge-gpu-virtual-machine-sizes.md#ncast4_v3-series-preview).

        ```json
            "vmSize": {
          "value": "Standard_NC4as_T4_v3"
        },
        ```

    1. После успешного создания виртуальной машины GPU эту виртуальную машину можно просмотреть в списке виртуальных машин в ресурсе Azure Stack ребра в портал Azure.

        ![Виртуальная машина GPU в списке виртуальных машин в портал Azure](media/azure-stack-edge-gpu-deploy-gpu-virtual-machine/list-virtual-machine-1.png)

1. Выберите виртуальную машину и перейдите к подробным сведениям. Скопируйте IP-адрес, выделенный для виртуальной машины.

    ![IP-адрес, выделенный для виртуальной машины GPU в портал Azure](media/azure-stack-edge-gpu-deploy-gpu-virtual-machine/get-ip-gpu-virtual-machine-1.png)

1. После создания виртуальной машины разверните расширение GPU с помощью шаблона расширения. Сведения об установке [расширения GPU для](#gpu-extension-for-linux) виртуальных машин Linux см. в [статье Установка расширения GPU для Windows](#gpu-extension-for-windows).

1. Чтобы проверить установку расширения GPU, подключитесь к виртуальной машине GPU:
    1. Если используется виртуальная машина Windows, выполните действия, описанные в разделе [Подключение к виртуальной машине Windows](azure-stack-edge-gpu-deploy-virtual-machine-powershell.md#connect-to-a-windows-vm). [Проверьте установку](#verify-windows-driver-installation).
    1. Если используется виртуальная машина Linux, выполните действия, описанные в статье [Подключение к виртуальной машине Linux](azure-stack-edge-gpu-deploy-virtual-machine-powershell.md#connect-to-a-linux-vm). [Проверьте установку](#verify-linux-driver-installation).

1. При необходимости можно переключить вычисленную сеть на любую необходимую. 


> [!NOTE]
> При обновлении версии программного обеспечения устройства с 2012 до более поздней версии необходимо вручную отключить виртуальные машины GPU.


## <a name="install-gpu-extension"></a>Установить расширение GPU

В зависимости от операционной системы для виртуальной машины можно установить расширение GPU для Windows или Linux.

> [!NOTE]
> Перед установкой расширения GPU убедитесь, что порт, включенный для среды выполнения вычислений на устройстве, подключен к Интернету и имеет доступ. Драйверы GPU загружаются через Интернет-доступ.

### <a name="gpu-extension-for-windows"></a>Расширение GPU для Windows

Чтобы развернуть драйверы NVIDIA GPU для существующей виртуальной машины, измените `addGPUExtWindowsVM.parameters.json` файл параметров, а затем разверните шаблон `addGPUextensiontoVM.json` .

#### <a name="edit-parameters-file"></a>Изменение файла параметров

Файл `addGPUExtWindowsVM.parameters.json` принимает следующие параметры:

```json
"parameters": { 
    "vmName": {
    "value": "<name of the VM>" 
    },
    "extensionName": {
    "value": "<name for the extension. Example: windowsGpu>" 
    },
    "publisher": {
    "value": "Microsoft.HpcCompute" 
    },
    "type": {
    "value": "NvidiaGpuDriverWindows" 
    },
    "typeHandlerVersion": {
    "value": "1.3" 
    },
    "settings": {
    "value": {
    "DriverURL" : "http://us.download.nvidia.com/tesla/442.50/442.50-tesla-desktop-winserver-2019-2016-international.exe",
    "DriverCertificateUrl" : "https://go.microsoft.com/fwlink/?linkid=871664",
    "DriverType":"CUDA"
    }
    }
    }
```

Ниже приведен пример файла параметров, который использовался в этой статье:

```powershell
PS C:\WINDOWS\system32> $templateFile = "C:\12-09-2020\CreateVM\CreateVM.json"
PS C:\WINDOWS\system32> $templateParameterFile = "C:\12-09-2020\CreateVM\CreateVM.parameters.json"
PS C:\WINDOWS\system32> $RGName = "myasegpuvm1"
PS C:\WINDOWS\system32> New-AzureRmResourceGroupDeployment -ResourceGroupName $RGName -TemplateFile $templateFile -TemplateParameterFile $templateParameterFile -Name "deployment2"

DeploymentName          : deployment2
ResourceGroupName       : myasegpuvm1
ProvisioningState       : Succeeded
Timestamp               : 12/16/2020 12:02:56 AM
Mode                    : Incremental
TemplateLink            :
Parameters              :
                          Name             Type                       Value
                          ===============  =========================  ==========
                          vmName           String                     VM2
                          adminUsername    String                     Administrator
                          password         String                     Password1
                          imageName        String                     myasewindowsimg
                          vmSize           String                     Standard_NC4as_T4_v3
                          vnetName         String                     ASEVNET
                          vnetRG           String                     aserg
                          subnetName       String                     ASEVNETsubNet
                          nicName          String                     nic6
                          ipConfigName     String                     ipconfig6
                          privateIPAddress  String

Outputs                 :
DeploymentDebugLogLevel :
PS C:\WINDOWS\system32>
```
#### <a name="deploy-template"></a>Развертывание шаблона

Разверните шаблон `addGPUextensiontoVM.json` . Этот шаблон развертывает расширение на существующей виртуальной машине. Выполните следующую команду:

```powershell
$templateFile = "<Path to addGPUextensiontoVM.json>" 
$templateParameterFile = "<Path to addGPUExtWindowsVM.parameters.json>" 
$RGName = "<Name of your resource group>"
New-AzureRmResourceGroupDeployment -ResourceGroupName $RGName -TemplateFile $templateFile -TemplateParameterFile $templateParameterFile -Name "<Name for your deployment>"
```
> [!NOTE]
> Развертывание расширения — это длительное задание, выполнение которого занимает около 10 минут.

Пример выходных данных:

```powershell
PS C:\WINDOWS\system32> "C:\12-09-2020\ExtensionTemplates\addGPUextensiontoVM.json"
C:\12-09-2020\ExtensionTemplates\addGPUextensiontoVM.json
PS C:\WINDOWS\system32> $templateFile = "C:\12-09-2020\ExtensionTemplates\addGPUextensiontoVM.json"
PS C:\WINDOWS\system32> $templateParameterFile = "C:\12-09-2020\ExtensionTemplates\addGPUExtWindowsVM.parameters.json"
PS C:\WINDOWS\system32> $RGName = "myasegpuvm1"
PS C:\WINDOWS\system32> New-AzureRmResourceGroupDeployment -ResourceGroupName $RGName -TemplateFile $templateFile -TemplateParameterFile $templateParameterFile -Name "deployment3"

DeploymentName          : deployment3
ResourceGroupName       : myasegpuvm1
ProvisioningState       : Succeeded
Timestamp               : 12/16/2020 12:18:50 AM
Mode                    : Incremental
TemplateLink            :
Parameters              :
                          Name             Type                       Value
                          ===============  =========================  ==========
                          vmName           String                     VM2
                          extensionName    String                     windowsgpuext
                          publisher        String                     Microsoft.HpcCompute
                          type             String                     NvidiaGpuDriverWindows
                          typeHandlerVersion  String                     1.3
                          settings         Object                     {
                            "DriverURL": "http://us.download.nvidia.com/tesla/442.50/442.50-tesla-desktop-winserver-2019-2016-international.exe",
                            "DriverCertificateUrl": "https://go.microsoft.com/fwlink/?linkid=871664",
                            "DriverType": "CUDA"
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
Пример выходных данных:

```powershell
PS C:\WINDOWS\system32> Get-AzureRmVMExtension -ResourceGroupName myasegpuvm1 -VMName VM2 -Name windowsgpuext

ResourceGroupName       : myasegpuvm1
VMName                  : VM2
Name                    : windowsgpuext
Location                : dbelocal
Etag                    : null
Publisher               : Microsoft.HpcCompute
ExtensionType           : NvidiaGpuDriverWindows
TypeHandlerVersion      : 1.3
Id                      : /subscriptions/947b3cfd-7a1b-4a90-7cc5-e52caf221332/resourceGroups/myasegpuvm1/providers/Microsoft.Compute/virtualMachines/VM2/extensions/windowsgpuext
PublicSettings          : {
                            "DriverURL": "http://us.download.nvidia.com/tesla/442.50/442.50-tesla-desktop-winserver-2019-2016-international.exe",
                            "DriverCertificateUrl": "https://go.microsoft.com/fwlink/?linkid=871664",
                            "DriverType": "CUDA"
                          }
ProtectedSettings       :
ProvisioningState       : Creating
Statuses                :
SubStatuses             :
AutoUpgradeMinorVersion : True
ForceUpdateTag          :

PS C:\WINDOWS\system32>
```

Выходные данные выполнения расширения записываются в следующий файл. Сведения `C:\Packages\Plugins\Microsoft.HpcCompute.NvidiaGpuDriverWindows\1.3.0.0\Status` об отслеживании состояния установки см. в этом файле. 


Успешная установка указывается `message` как `Enable Extension` и `status` `success` .

```powershell
"status":  {
                       "formattedMessage":  {
                                                "message":  "Enable Extension",
                                                "lang":  "en"
                                            },
                       "name":  "NvidiaGpuDriverWindows",
                       "status":  "success",
```

#### <a name="verify-windows-driver-installation"></a>Проверка установки драйвера Windows

Войдите на виртуальную машину и запустите служебную программу командной строки NVIDIA-SMI, установленную вместе с драйвером. Находится `nvidia-smi.exe` в папке  `C:\Program Files\NVIDIA Corporation\NVSMI\nvidia-smi.exe` . Если файл не отображается, возможно, установка драйвера выполняется в фоновом режиме. Подождите 10 минут и снова проверьте.

Если драйвер установлен, отобразятся выходные данные, аналогичные приведенным в следующем примере: 

```powershell
PS C:\Users\Administrator> cd "C:\Program Files\NVIDIA Corporation\NVSMI"
PS C:\Program Files\NVIDIA Corporation\NVSMI> ls

    Directory: C:\Program Files\NVIDIA Corporation\NVSMI

Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        2/26/2020  12:00 PM         849640 MCU.exe
-a----        2/26/2020  12:00 PM         443104 nvdebugdump.exe
-a----        2/25/2020   2:06 AM          81823 nvidia-smi.1.pdf
-a----        2/26/2020  12:01 PM         566880 nvidia-smi.exe
-a----        2/26/2020  12:01 PM         991344 nvml.dll

PS C:\Program Files\NVIDIA Corporation\NVSMI> .\nvidia-smi.exe
Wed Dec 16 00:35:51 2020
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 442.50       Driver Version: 442.50       CUDA Version: 10.2     |
|-------------------------------+----------------------+----------------------+
| GPU  Name            TCC/WDDM | Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|===============================+======================+======================|
|   0  Tesla T4            TCC  | 0000503C:00:00.0 Off |                    0 |
| N/A   35C    P8    11W /  70W |      8MiB / 15205MiB |      0%      Default |
+-------------------------------+----------------------+----------------------+

+-----------------------------------------------------------------------------+
| Processes:                                                       GPU Memory |
|  GPU       PID   Type   Process name                             Usage      |
|=============================================================================|
|  No running processes found                                                 |
+-----------------------------------------------------------------------------+
PS C:\Program Files\NVIDIA Corporation\NVSMI>
```

Дополнительные сведения см. в статье [расширение драйвера GPU NVIDIA для Windows](../virtual-machines/extensions/hpccompute-gpu-windows.md).

### <a name="gpu-extension-for-linux"></a>Расширение GPU для Linux

Чтобы развернуть драйверы NVIDIA GPU для существующей виртуальной машины, измените файл параметров, а затем разверните шаблон `addGPUextensiontoVM.json` . Существуют определенные файлы параметров для Ubuntu и Red Hat Enterprise Linux (RHEL), как описано в следующих разделах.

#### <a name="edit-parameters-file"></a>Изменение файла параметров

При использовании Ubuntu `addGPUExtLinuxVM.parameters.json` файл принимает следующие параметры:

```powershell
"parameters": { 
    "vmName": {
    "value": "<name of the VM>" 
    },
    "extensionName": {
    "value": "<name for the extension. Example: linuxGpu>" 
    },
    "publisher": {
    "value": "Microsoft.HpcCompute" 
    },
    "type": {
    "value": "NvidiaGpuDriverLinux" 
    },
    "typeHandlerVersion": {
    "value": "1.3" 
    },
    "settings": {
    "value": {
    "DRIVER_URL": "https://go.microsoft.com/fwlink/?linkid=874271",
    "PUBKEY_URL": "http://download.microsoft.com/download/F/F/A/FFAC979D-AD9C-4684-A6CE-C92BB9372A3B/7fa2af80.pub",
    "CUDA_ver": "10.0.130",
    "InstallCUDA": "true"
    }
    }
    }
```
При использовании Red Hat Enterprise Linux (RHEL) `addGPUExtensionRHELVM.parameters.json` файл принимает следующие параметры:

```powershell
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "vmName": {
            "value": "<name of the VM>" 
        },
        "extensionName": {
            "value": "<name for the extension. Example: linuxGpu>" 
        },
        "publisher": {
            "value": "Microsoft.HpcCompute" 
        },
        "type": {
            "value": "NvidiaGpuDriverLinux" 
        },
        "typeHandlerVersion": {
            "value": "1.3" 
        },
        "settings": {
            "value": {
                    "isCustomInstall":true,
                    "DRIVER_URL":"https://go.microsoft.com/fwlink/?linkid=874273",
                    "CUDA_ver":"10.0.130",
                    "PUBKEY_URL":"http://download.microsoft.com/download/F/F/A/FFAC979D-AD9C-4684-A6CE-C92BB9372A3B/7fa2af80.pub",
                    "DKMS_URL":"https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm",
                    "LIS_URL":"https://aka.ms/lis",
                    "LIS_RHEL_ver":"3.10.0-1062.9.1.el7"
            }
        }
    }
}
```


Ниже приведен пример файла параметров Ubuntu, который использовался в этой статье:

```powershell
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "vmName": {
            "value": "VM1" 
        },
        "extensionName": {
            "value": "gpuLinux" 
        },
        "publisher": {
            "value": "Microsoft.HpcCompute" 
        },
        "type": {
            "value": "NvidiaGpuDriverLinux" 
        },
        "typeHandlerVersion": {
            "value": "1.3" 
        },
        "settings": {
            "value": {
            "DRIVER_URL": "https://go.microsoft.com/fwlink/?linkid=874271",
            "PUBKEY_URL": "http://download.microsoft.com/download/F/F/A/FFAC979D-AD9C-4684-A6CE-C92BB9372A3B/7fa2af80.pub",
            "CUDA_ver": "10.0.130",
            "InstallCUDA": "true"
            }
        }
    }
}
```


#### <a name="deploy-template"></a>Развертывание шаблона

Разверните шаблон `addGPUextensiontoVM.json` . Этот шаблон развертывает расширение на существующей виртуальной машине. Выполните следующую команду:

```powershell
$templateFile = "Path to addGPUextensiontoVM.json" 
$templateParameterFile = "Path to addGPUExtLinuxVM.parameters.json" 
$RGName = "<Name of your resource group>" 
New-AzureRmResourceGroupDeployment -ResourceGroupName $RGName -TemplateFile $templateFile -TemplateParameterFile $templateParameterFile -Name "<Name for your deployment>"
``` 

> [!NOTE]
> Развертывание расширения — это длительное задание, выполнение которого занимает около 10 минут.

Пример выходных данных:

```powershell
Copyright (C) Microsoft Corporation. All rights reserved.
Try the new cross-platform PowerShell https://aka.ms/pscore6

PS C:\WINDOWS\system32> $templateFile = "C:\12-09-2020\ExtensionTemplates\addGPUextensiontoVM.json"
PS C:\WINDOWS\system32> $templateParameterFile = "C:\12-09-2020\ExtensionTemplates\addGPUExtLinuxVM.parameters.json"
PS C:\WINDOWS\system32> $RGName = "rg2"
PS C:\WINDOWS\system32> New-AzureRmResourceGroupDeployment -ResourceGroupName $RGName -TemplateFile $templateFile -TemplateParameterFile $templateParameterFile -Name "delpoyment7"

DeploymentName          : delpoyment7
ResourceGroupName       : rg2
ProvisioningState       : Succeeded
Timestamp               : 12/10/2020 10:43:23 PM
Mode                    : Incremental
TemplateLink            :
Parameters              :
                          Name             Type                       Value
                          ===============  =========================  ==========
                          vmName           String                     VM1
                          extensionName    String                     gpuLinux
                          publisher        String                     Microsoft.HpcCompute
                          type             String                     NvidiaGpuDriverLinux
                          typeHandlerVersion  String                     1.3
                          settings         Object                     {
                            "DRIVER_URL": "https://go.microsoft.com/fwlink/?linkid=874271",
                            "PUBKEY_URL":
                          "http://download.microsoft.com/download/F/F/A/FFAC979D-AD9C-4684-A6CE-C92BB9372A3B/7fa2af80.pub",
                            "CUDA_ver": "10.0.130",
                            "InstallCUDA": "true"
                          }

Outputs                 :
DeploymentDebugLogLevel :
PS C:\WINDOWS\system32>
```

#### <a name="track-deployment-status"></a>Отслеживание состояния развертывания    
    
Шаблоны развертывания является долго выполняющимся заданием. Чтобы проверить состояние развертывания расширений для данной виртуальной машины, откройте другой сеанс PowerShell (Запуск от имени администратора). Выполните следующую команду: 

```powershell
Get-AzureRmVMExtension -ResourceGroupName myResourceGroup -VMName <VM Name> -Name <Extension Name>
```
Пример выходных данных: 

```powershell
Copyright (C) Microsoft Corporation. All rights reserved.
Try the new cross-platform PowerShell https://aka.ms/pscore6

PS C:\WINDOWS\system32> Get-AzureRmVMExtension -ResourceGroupName rg2 -VMName VM1 -Name gpulinux

ResourceGroupName       : rg2
VMName                  : VM1
Name                    : gpuLinux
Location                : dbelocal
Etag                    : null
Publisher               : Microsoft.HpcCompute
ExtensionType           : NvidiaGpuDriverLinux
TypeHandlerVersion      : 1.3
Id                      : /subscriptions/947b3cfd-7a1b-4a90-7cc5-e52caf221332/resourceGroups/rg2/providers/Microsoft.Compute/virtualMachines/VM1/extensions/gpuLinux
PublicSettings          : {
                            "DRIVER_URL": "https://go.microsoft.com/fwlink/?linkid=874271",
                            "PUBKEY_URL": "http://download.microsoft.com/download/F/F/A/FFAC979D-AD9C-4684-A6CE-C92BB9372A3B/7fa2af80.pub",
                            "CUDA_ver": "10.0.130",
                            "InstallCUDA": "true"
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

Выходные данные выполнения расширения записываются в следующий файл: `/var/log/azure/nvidia-vmext-status` .

#### <a name="verify-linux-driver-installation"></a>Проверка установки драйвера Linux

Чтобы проверить установку драйвера, выполните следующие действия.

1. Подключитесь к виртуальной машине GPU. Следуйте инструкциям в статье [Подключение к виртуальной машине Linux](azure-stack-edge-gpu-deploy-virtual-machine-powershell.md#connect-to-a-linux-vm). 

    Пример выходных данных:

    ```powershell
    PS C:\WINDOWS\system32> ssh -l Administrator 10.57.50.60
    Administrator@10.57.50.60's password:
    Welcome to Ubuntu 18.04.4 LTS (GNU/Linux 5.0.0-1031-azure x86_64)
     * Documentation:  https://help.ubuntu.com
     * Management:     https://landscape.canonical.com
     * Support:        https://ubuntu.com/advantage
      System information as of Thu Dec 10 22:57:01 UTC 2020
    
      System load:  0.0                Processes:           133
      Usage of /:   24.8% of 28.90GB   Users logged in:     0
      Memory usage: 2%                 IP address for eth0: 10.57.50.60
      Swap usage:   0%
    
    249 packages can be updated.
    140 updates are security updates.
    
    Welcome to Ubuntu 18.04.4 LTS (GNU/Linux 5.0.0-1031-azure x86_64)    
     * Documentation:  https://help.ubuntu.com
     * Management:     https://landscape.canonical.com
     * Support:        https://ubuntu.com/advantage    
      System information as of Thu Dec 10 22:57:01 UTC 2020    
      System load:  0.0                Processes:           133
      Usage of /:   24.8% of 28.90GB   Users logged in:     0
      Memory usage: 2%                 IP address for eth0: 10.57.50.60
      Swap usage:   0%
        
    249 packages can be updated.
    140 updates are security updates.
    
    New release '20.04.1 LTS' available.
    Run 'do-release-upgrade' to upgrade to it.
        
    *** System restart required ***
    Last login: Thu Dec 10 21:49:29 2020 from 10.90.24.23
    To run a command as administrator (user "root"), use "sudo <command>".
    See "man sudo_root" for details.
    
    Administrator@VM1:~$
    ```
2. Запустите служебную программу командной строки NVIDIA-SMI, установленную вместе с драйвером. Если драйвер успешно установлен, вы сможете запустить программу и просмотреть следующие выходные данные:

    ```powershell
    Administrator@VM1:~$ nvidia-smi
    Thu Dec 10 22:58:46 2020
    +-----------------------------------------------------------------------------+
    | NVIDIA-SMI 455.45.01    Driver Version: 455.45.01    CUDA Version: 11.1     |
    |-------------------------------+----------------------+----------------------+
    | GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
    | Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
    |                               |                      |               MIG M. |
    |===============================+======================+======================|
    |   0  Tesla T4            Off  | 0000941F:00:00.0 Off |                    0 |
    | N/A   48C    P0    27W /  70W |      0MiB / 15109MiB |      5%      Default |
    |                               |                      |                  N/A |
    +-------------------------------+----------------------+----------------------+
    
    +-----------------------------------------------------------------------------+
    | Processes:                                                                  |
    |  GPU   GI   CI        PID   Type   Process name                  GPU Memory |
    |        ID   ID                                                   Usage      |
    |=============================================================================|
    |  No running processes found                                                 |
    +-----------------------------------------------------------------------------+
    Administrator@VM1:~$
    ```

Дополнительные сведения см. в статье [расширение драйвера GPU NVIDIA для Linux](../virtual-machines/extensions/hpccompute-gpu-linux.md).

## <a name="remove-gpu-extension"></a>Удалить расширение GPU

Чтобы удалить расширение GPU, используйте следующую команду:

`Remove-AzureRmVMExtension -ResourceGroupName <Resource group name> -VMName <VM name> -Name <Extension name>`

Пример выходных данных:

```powershell
PS C:\azure-stack-edge-deploy-vms> Remove-AzureRmVMExtension -ResourceGroupName rgl -VMName WindowsVM -Name windowsgpuext
Virtual machine extension removal operation
This cmdlet will remove the specified virtual machine extension. Do you want to continue? [Y] Yes [N] No [S] Suspend [?] Help (default is "Y"): y
Requestld IsSuccessStatusCode StatusCode ReasonPhrase
--------- ------------------- ---------- ------------    
          True                OK         OK
```




## <a name="next-steps"></a>Дальнейшие действия

[Командлеты Azure Resource Manager](/powershell/module/azurerm.resources/?view=azurermps-6.13.0&preserve-view=true)