---
title: Распространенные ошибки при миграции из классической модели в модель Azure Resource Manager
description: В этой статье перечислены наиболее распространенные ошибки и способы их устранения при переносе ресурсов IaaS из службы управления службами Azure в Azure Resource Manager.
author: tanmaygore
manager: vashan
ms.service: virtual-machines
ms.subservice: classic-to-arm-migration
ms.workload: infrastructure-services
ms.topic: troubleshooting
ms.date: 02/06/2020
ms.author: tagore
ms.openlocfilehash: 6d803d1a66c069f5eb42deead453a8526577f76b
ms.sourcegitcommit: d135e9a267fe26fbb5be98d2b5fd4327d355fe97
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102615216"
---
# <a name="errors-that-commonly-occur-during-classic-to-azure-resource-manager-migration"></a>Ошибки, которые обычно возникают во время классической модели для Azure Resource Manager миграции

> [!IMPORTANT]
> Сегодня около 90% виртуальных машин IaaS используют [Azure Resource Manager](https://azure.microsoft.com/features/resource-manager/). По состоянию на 28 февраля 2020, классические виртуальные машины являются устаревшими и будут полностью сняты с 1 марта 2023. [Узнайте больше]( https://aka.ms/classicvmretirement) об этой нерекомендуемости и [о том, как она влияет на вас](classic-vm-deprecation.md#how-does-this-affect-me).

В этой статье перечислены самые распространенные ошибки при переносе ресурсов IaaS из классической модели развертывания Azure в стек Azure Resource Manager, а также способы их устранения.


## <a name="list-of-errors"></a>Список ошибок

| Строка ошибки | Меры по снижению риска |
| --- | --- |
| Внутренняя ошибка сервера |В некоторых случаях это временная ошибка, которая исчезает после повторной попытки. Если она не исчезает, [обратитесь в службу поддержки Azure](../azure-portal/supportability/how-to-create-azure-support-request.md), так как в этом случае требуется исследование журналов платформы. <br><br> **ПРИМЕЧАНИЕ.** Как только группа поддержки приступит к отслеживанию инцидента, не пытайтесь устранить проблему самостоятельно, так как это может иметь непредвиденные последствия для вашей среды. |
| Миграция не поддерживается для развертывания {deployment-name} в размещенной службе {hosted-service-name}, так как это развертывание PaaS (веб-роль или рабочая роль). |Это происходит, если развертывание содержит веб-роль или рабочую роль. Так как миграция поддерживается только для виртуальных машин, удалите веб-роль и рабочую роль в развертывании и выполните миграцию снова. |
| Не удалось выполнить развертывание шаблона {template-name}. CorrelationId={guid} |В серверной части службы миграции мы используем шаблоны Azure Resource Manager для создания ресурсов в стеке Azure Resource Manager. Так как шаблоны являются идемпотентными, обычно можно безопасно повторить операцию миграции для устранения этой ошибки. Если эта ошибка не исчезнет, [обратитесь в службу поддержки Azure](../azure-portal/supportability/how-to-create-azure-support-request.md) и предоставьте CorrelationId. <br><br> **ПРИМЕЧАНИЕ.** Как только группа поддержки приступит к отслеживанию инцидента, не пытайтесь устранить проблему самостоятельно, так как это может иметь непредвиденные последствия для вашей среды. |
| Виртуальная сеть {virtual-network-name} не существует. |Это может произойти, если вы создали виртуальную сеть на новом портале Azure. Фактическое имя виртуальной сети соответствует шаблону "группа * \<VNET name>". |
| Виртуальная машина {vm-name} в размещенной службе {hosted-service-name} содержит расширение {extension-name}, которое не поддерживается в Azure Resource Manager. Рекомендуется удалить его из виртуальной машины перед продолжением миграции. |Расширения XML, такие как BGInfo 1. \* не поддерживаются в Azure Resource Manager. Таким образом их невозможно перенести. Если оставить эти расширения на виртуальной машине, они автоматически удаляются перед выполнением миграции. |
| Виртуальная машина {vm-name} в размещенной службе {hosted-service-name} содержит расширение VMSnapshot или VMSnapshotLinux, которое сейчас не поддерживается для миграции. Удалите его с виртуальной машины и добавьте снова с помощью Azure Resource Manager после завершения миграции. |В этом сценарии виртуальная машина настроена для службы архивации Azure. Так как в настоящее время этот сценарий не поддерживается, используйте решение по адресу https://aka.ms/vmbackupmigration. |
| Виртуальная машина {vm-name} в размещенной службе {hosted-service-name} содержит расширение {extension-name}, о состоянии которого она не сообщает. Поэтому виртуальную машину нельзя перенести. Убедитесь, что о состоянии расширения сообщается, или удалите расширение с виртуальной машины и повторите миграцию. <br><br> Виртуальная машина {vm-name} в размещенной службе {hosted-service-name} содержит расширение {extension-name}, сообщающее о состоянии обработчика: {handler-status}. Hence, the VM cannot be migrated. Убедитесь, что сообщаемое состояние обработчика — {handler-status}, или удалите расширение с виртуальной машины и повторите миграцию. <br><br> Агент виртуальной машины {vm-name} в размещенной службе {hosted-service-name} сообщает, что общее состояние агента — Not Ready. Таким образом виртуальную машину невозможно перенести, если она содержит расширение, которое можно перенести. Убедитесь, что агент виртуальной машины сообщает о том, что общее состояние агента — Ready. См. https://aka.ms/classiciaasmigrationfaqs. |Гостевому агенту Azure и расширениям виртуальных машин требуется исходящий доступ к Интернету для учетной записи хранения виртуальной машины, чтобы заполнить данные о состоянии. Стандартные причины сбоя состояния: <li> группа безопасности сети, блокирующая исходящий доступ к Интернету; <li> Если виртуальная сеть находится на локальных DNS-серверах и потеряна связь с DNS <br><br> Если сообщение о неподдерживаемом состоянии не исчезнет, можно удалить расширения, чтобы пропустить эту проверку и перейти к миграции. |
| Миграция не поддерживается для развертывания {deployment-name} в размещенной службе {hosted-service-name}, так как она содержит несколько наборов доступности. |В настоящее время можно перенести только размещенные службы с одной и менее группами доступности. Чтобы обойти эту проблему, переместите дополнительные наборы доступности и виртуальные машины в этих группах доступности в другую размещенную службу. |
| Миграция не поддерживается для развертывания {deployment-name} в размещенной службе {hosted-service-name}, так как она содержит виртуальные машины, которые не входят в группу доступности, но содержатся в размещенной службе. |В этом сценарии нужно переместить все виртуальные машины в одну группу доступности или удалить их из группы доступности в размещенной службе. |
| Учетная запись хранения, размещенная служба или виртуальная сеть {virtual-network-name} переносится и поэтому ее невозможно изменить. |Эта ошибка возникает, после завершения операции подготовки к миграции для ресурса и инициации операции изменения ресурса. Так как плоскость управления блокируется после операции подготовки, любые изменения ресурса также блокируются. Чтобы разблокировать плоскость управления, можно запустить операцию фиксации миграции, чтобы завершить ее, или прерывания, чтобы сделать откат операции подготовки. |
| Миграция запрещена для размещенной службы {размещенных service-name}, так как она содержит виртуальную машину {vm-name} в состоянии RoleStateUnknown. Миграцию можно выполнить, только если виртуальная машина находится в одном из следующих состояний: Running, Stopped, Stopped Deallocated. |Виртуальная машина может проходить переход между состояниями, что обычно происходит при выполнении операции обновления HostedService, такой как перезагрузка, установка расширения и т. д. Перед выполнением миграции рекомендуется выполнить операцию обновления в HostedService. |
| Развертывание {deployment-name} в размещенной службе {hosted-service-name} содержит виртуальную машину {vm-name} с диском данных {data-disk-name}, у которого физический размер большого двоичного объекта {size-of-the-vhd-blob-backing-the-data-disk} в байтах не соответствует логическому размеру диска данных виртуальной машины {size-of-the-data-disk-specified-in-the-vm-api}. Миграция будет выполнена без указания размера диска данных для виртуальной машины Azure Resource Manager. | Эта ошибка возникает, если размер большого двоичного объекта VHD изменяется без обновления размера в модели API виртуальной машины. Подробное описание шагов по устранению этой ошибки приводится [ниже](#vm-with-data-disk-whose-physical-blob-size-bytes-does-not-match-the-vm-data-disk-logical-size-bytes).|
| При проверке диска данных {имя диска данных} возникло исключение хранилища: ссылка на носитель {универсальный код ресурса (URI) диска данных} виртуальной машины {имя виртуальной машины} в облачной службе {имя облачной службы}. Убедитесь, что ссылка на носитель VHD для данной виртуальной машины доступна. | Эта ошибка может произойти, если диски виртуальной машины были удалены или больше недоступны. Убедитесь, что диски виртуальной машины существуют.|
| Виртуальная машина {имя_ВМ} в HostedService {имя_облачной_службы} содержит диск с MediaLink {URI_VHD} с именем BLOB-объекта {имя_объекта_VHD}, которое не поддерживается в Azure Resource Manager. | Эта ошибка возникает, когда имя большого двоичного объекта содержит символ "/", который в настоящее время не поддерживается поставщиком вычислительных ресурсов. |
| Миграция запрещена для развертывания {имя_развертывания} в размещенной службе {имя_облачной_службы}, так как она не входит в область для региона. Чтобы \/ переместить это развертывание в региональную область, обратитесь по адресу https:/AKA.MS/regionalscope. | В 2014 г. было объявлено, что сетевые ресурсы в Azure будут перемещены из области работ для кластера на региональный уровень. [https://aka.ms/regionalscope](https://aka.ms/regionalscope)Дополнительные сведения см. в разделе. Эта ошибка возникает при миграции развертывания без выполнения обновления, которое автоматически перемещает такое развертывание в область для региона. Лучше всего добавить конечную точку в виртуальную машину или диск данных в виртуальную машину, а затем повторить попытку миграции. <br> Узнайте, [как настроить конечные точки на классической виртуальной машине в Azure](/previous-versions/azure/virtual-machines/windows/classic/setup-endpoints#create-an-endpoint) или [подключить диск данных к виртуальной машине, созданной с помощью классической модели развертывания](./linux/attach-disk-portal.md) .|
| Миграция не поддерживается для виртуальной сети {vnet-name}, так как она включает в себя развертывания PaaS без шлюза. | Эта ошибка возникает при наличии развертываний PaaS без шлюза, например шлюза приложений или служб управления API, которые подключены к виртуальной сети.|


## <a name="detailed-mitigations"></a>Подробные инструкции по устранению

### <a name="vm-with-data-disk-whose-physical-blob-size-bytes-does-not-match-the-vm-data-disk-logical-size-bytes"></a>Виртуальная машина с диском данных, у которого физический размер большого двоичного объекта в байтах не соответствует логическому размеру диска данных виртуальной машины.

Это может произойти, когда нарушается синхронизация логического размера диска данных с фактическим размером большого двоичного объекта VHD. Это можно легко проверить с помощью следующих команд:

#### <a name="verifying-the-issue"></a>Проверка наличия проблемы

```powershell
# Store the VM details in the VM object
$vm = Get-AzureVM -ServiceName $servicename -Name $vmname

# Display the data disk properties
# NOTE the data disk LogicalDiskSizeInGB below which is 11GB. Also note the MediaLink Uri of the VHD blob as we'll use this in the next step
$vm.VM.DataVirtualHardDisks


HostCaching         : None
DiskLabel           : 
DiskName            : coreosvm-coreosvm-0-201611230636240687
Lun                 : 0
LogicalDiskSizeInGB : 11
MediaLink           : https://contosostorage.blob.core.windows.net/vhds/coreosvm-dd1.vhd
SourceMediaLink     : 
IOType              : Standard
ExtensionData       : 

# Now get the properties of the blob backing the data disk above
# NOTE the size of the blob is about 15 GB which is different from LogicalDiskSizeInGB above
$blob = Get-AzStorageblob -Blob "coreosvm-dd1.vhd" -Container vhds 

$blob

ICloudBlob        : Microsoft.WindowsAzure.Storage.Blob.CloudPageBlob
BlobType          : PageBlob
Length            : 16106127872
ContentType       : application/octet-stream
LastModified      : 11/23/2016 7:16:22 AM +00:00
SnapshotTime      : 
ContinuationToken : 
Context           : Microsoft.WindowsAzure.Commands.Common.Storage.AzureStorageContext
Name              : coreosvm-dd1.vhd
```

#### <a name="mitigating-the-issue"></a>Устранение проблемы

```powershell
# Convert the blob size in bytes to GB into a variable which we'll use later
$newSize = [int]($blob.Length / 1GB)

# See the calculated size in GB
$newSize

15

# Store the disk name of the data disk as we'll use this to identify the disk to be updated
$diskName = $vm.VM.DataVirtualHardDisks[0].DiskName

# Identify the LUN of the data disk to remove
$lunToRemove = $vm.VM.DataVirtualHardDisks[0].Lun

# Now remove the data disk from the VM so that the disk isn't leased by the VM and it's size can be updated
Remove-AzureDataDisk -LUN $lunToRemove -VM $vm | Update-AzureVm -Name $vmname -ServiceName $servicename

OperationDescription OperationId                          OperationStatus
-------------------- -----------                          ---------------
Update-AzureVM       213xx1-b44b-1v6n-23gg-591f2a13cd16   Succeeded  

# Verify we have the right disk that's going to be updated
Get-AzureDisk -DiskName $diskName

AffinityGroup        : 
AttachedTo           : 
IsCorrupted          : False
Label                : 
Location             : East US
DiskSizeInGB         : 11
MediaLink            : https://contosostorage.blob.core.windows.net/vhds/coreosvm-dd1.vhd
DiskName             : coreosvm-coreosvm-0-201611230636240687
SourceImageName      : 
OS                   : 
IOType               : Standard
OperationDescription : Get-AzureDisk
OperationId          : 0c56a2b7-a325-123b-7043-74c27d5a61fd
OperationStatus      : Succeeded

# Now update the disk to the new size
Update-AzureDisk -DiskName $diskName -ResizedSizeInGB $newSize -Label $diskName

OperationDescription OperationId                          OperationStatus
-------------------- -----------                          ---------------
Update-AzureDisk     cv134b65-1b6n-8908-abuo-ce9e395ac3e7 Succeeded 

# Now verify that the "DiskSizeInGB" property of the disk matches the size of the blob 
Get-AzureDisk -DiskName $diskName


AffinityGroup        : 
AttachedTo           : 
IsCorrupted          : False
Label                : coreosvm-coreosvm-0-201611230636240687
Location             : East US
DiskSizeInGB         : 15
MediaLink            : https://contosostorage.blob.core.windows.net/vhds/coreosvm-dd1.vhd
DiskName             : coreosvm-coreosvm-0-201611230636240687
SourceImageName      : 
OS                   : 
IOType               : Standard
OperationDescription : Get-AzureDisk
OperationId          : 1v53bde5-cv56-5621-9078-16b9c8a0bad2
OperationStatus      : Succeeded

# Now we'll add the disk back to the VM as a data disk. First we need to get an updated VM object
$vm = Get-AzureVM -ServiceName $servicename -Name $vmname

Add-AzureDataDisk -Import -DiskName $diskName -LUN 0 -VM $vm -HostCaching ReadWrite | Update-AzureVm -Name $vmname -ServiceName $servicename

OperationDescription OperationId                          OperationStatus
-------------------- -----------                          ---------------
Update-AzureVM       b0ad3d4c-4v68-45vb-xxc1-134fd010d0f8 Succeeded      
```

### <a name="moving-a-vm-to-a-different-subscription-after-completing-migration"></a>Перемещение виртуальной машины в другую подписку после миграции

Возможно, после завершения миграции вы решите переместить виртуальную машину в другую подписку. Тем не менее, если на виртуальной машине имеется секрет или ключ, который ссылается на ресурс Key Vault, то переместить ее в настоящее время не удастся. Приведенные ниже инструкции позволяют решить эту задачу. 

#### <a name="powershell"></a>PowerShell

```powershell
$vm = Get-AzVM -ResourceGroupName "MyRG" -Name "MyVM"
Remove-AzVMSecret -VM $vm
Update-AzVM -ResourceGroupName "MyRG" -VM $vm
```

#### <a name="azure-cli"></a>Azure CLI

```azurecli
az vm update -g "myrg" -n "myvm" --set osProfile.Secrets=[]
```


## <a name="next-steps"></a>Дальнейшие действия

* [Обзор поддерживаемого платформой переноса ресурсов IaaS из классической модели в модель Azure Resource Manager](migration-classic-resource-manager-overview.md)
* [Техническое руководство по поддерживаемому платформой переносу из классической модели в модель Azure Resource Manager](migration-classic-resource-manager-deep-dive.md)
* [Planning for migration of IaaS resources from classic to Azure Resource Manager](migration-classic-resource-manager-plan.md) (Планирование переноса ресурсов IaaS из классической модели в модель Azure Resource Manager)
* [Перенос ресурсов IaaS из классической модели в модель Azure Resource Manager с помощью Azure PowerShell](migration-classic-resource-manager-ps.md)
* [Перенос ресурсов IaaS из классической модели в модель Azure Resource Manager с помощью интерфейса командной строки Azure](migration-classic-resource-manager-cli.md)
* [Community tools to migrate IaaS resources from classic to Azure Resource Manager](migration-classic-resource-manager-community-tools.md) (Инструменты сообщества для перемещения ресурсов IaaS из классической модели в модель Azure Resource Manager)
* [Часто задаваемые вопросы о миграции из классической модели в модель Azure Resource Manager](migration-classic-resource-manager-faq.md)