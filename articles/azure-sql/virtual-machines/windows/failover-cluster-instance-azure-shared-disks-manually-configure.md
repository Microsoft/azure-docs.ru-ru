---
title: Создание FCI с общими дисками Azure
description: Используйте общие диски Azure для создания экземпляра отказоустойчивого кластера (FCI) с SQL Server на виртуальных машинах Azure.
services: virtual-machines
documentationCenter: na
author: MashaMSFT
editor: monicar
tags: azure-service-management
ms.service: virtual-machines-sql
ms.custom: na
ms.topic: how-to
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 06/26/2020
ms.author: mathoma
ms.openlocfilehash: ce77021e74507ead6d225081debc7024cb89a15a
ms.sourcegitcommit: f3ec73fb5f8de72fe483995bd4bbad9b74a9cc9f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102042408"
---
# <a name="create-an-fci-with-azure-shared-disks-sql-server-on-azure-vms"></a>Создание FCI с общими дисками Azure (SQL Server на виртуальных машинах Azure)
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

В этой статье объясняется, как создать экземпляр отказоустойчивого кластера (FCI) с помощью общих дисков Azure с SQL Server на виртуальных машинах Azure. 

Дополнительные сведения см. в обзоре [FCI с SQL Server на виртуальных машинах Azure и в](failover-cluster-instance-overview.md) разделе рекомендации по работе с [кластерами](hadr-cluster-best-practices.md). 

## <a name="prerequisites"></a>Предварительные требования 

Перед выполнением инструкций, приведенных в этой статье, у вас уже должны быть:

- Подписка Azure. Начните работу [бесплатно](https://azure.microsoft.com/free/). 
- [Две или более виртуальных машин Windows Azure](failover-cluster-instance-prepare-vm.md). [Наборы доступности](../../../virtual-machines/windows/tutorial-availability-sets.md) и [группы размещения](../../../virtual-machines/co-location.md#proximity-placement-groups) ппгс, поддерживаемые для SSD (цен. Категория "Премиум") и [зон доступности](../../../virtual-machines/windows/create-portal-availability-zone.md#confirm-zone-for-managed-disk-and-ip-address) , поддерживаются для Ultra-дисков. Все узлы должны находиться в одной [группе размещения близости](../../../virtual-machines/co-location.md#proximity-placement-groups).
- учетная запись с разрешениями на создание объектов как на виртуальных машинах Azure, так и в Active Directory;
- Последняя версия [PowerShell](/powershell/azure/install-az-ps). 

## <a name="add-azure-shared-disk"></a>Добавление общего диска Azure
Развертывание управляемого SSD (цен. категория "Премиум") диска с включенным компонентом "общий диск". Задайте `maxShares` для параметра значение **соответствует количеству узлов кластера** , чтобы сделать диск общим для всех узлов FCI. 

Добавьте общий диск Azure, выполнив следующие действия. 

1. Сохраните следующий скрипт как *SharedDiskConfig.js*: 

   ```JSON
   { 
     "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
     "contentVersion": "1.0.0.0",
     "parameters": {
       "dataDiskName": {
         "type": "string",
         "defaultValue": "mySharedDisk"
       },
       "dataDiskSizeGB": {
         "type": "int",
         "defaultValue": 1024
       },
       "maxShares": {
         "type": "int",
         "defaultValue": 2
       }
     },
     "resources": [
       {
         "type": "Microsoft.Compute/disks",
         "name": "[parameters('dataDiskName')]",
         "location": "[resourceGroup().location]",
         "apiVersion": "2019-07-01",
         "sku": {
           "name": "Premium_LRS"
         },
         "properties": {
           "creationData": {
             "createOption": "Empty"
           },
           "diskSizeGB": "[parameters('dataDiskSizeGB')]",
           "maxShares": "[parameters('maxShares')]"
         }
       }
     ] 
   }
   ```

2. Запустите *SharedDiskConfig.jsв* с помощью PowerShell: 

   ```powershell
   $rgName = < specify your resource group name>
       $location = 'westcentralus'
       New-AzResourceGroupDeployment -ResourceGroupName $rgName `
   -TemplateFile "SharedDiskConfig.json"
   ```

3. Для каждой виртуальной машины инициализируйте подключенные общие диски как таблицу разделов GUID (GPT) и отформатируйте их как новую технологию файловая система (NTFS), выполнив следующую команду: 

    ```powershell
    $resourceGroup = "<your resource group name>"
    $location = "<region of your shared disk>"
    $ppgName = "<your proximity placement groups name>"
    $vm = Get-AzVM -ResourceGroupName "<your resource group name>" `
        -Name "<your VM node name>"
    $dataDisk = Get-AzDisk -ResourceGroupName $resourceGroup `
        -DiskName "<your shared disk name>"
    $vm = Add-AzVMDataDisk -VM $vm -Name "<your shared disk name>" `
        -CreateOption Attach -ManagedDiskId $dataDisk.Id `
        -Lun <available LUN - check disk setting of the VM>
    Update-AzVm -VM $vm -ResourceGroupName $resourceGroup
    ```

## <a name="create-failover-cluster"></a>Создание отказоустойчивого кластера

Для создания отказоустойчивого кластера требуется следующее.

- Имена виртуальных машин, которые становятся узлами кластера.
- Имя отказоустойчивого кластера.
- IP-адрес отказоустойчивого кластера. Вы можете использовать IP-адрес, который не используется в той же виртуальной сети и подсети Azure, где расположены узлы кластера.

# <a name="windows-server-2012-2016"></a>[Windows Server 2012-2016](#tab/windows2012)

Следующий сценарий PowerShell создает отказоустойчивый кластер. Обновите скрипт, введя имена узлов (имена виртуальных машин) и доступный IP-адрес из виртуальной сети Azure.

```powershell
New-Cluster -Name <FailoverCluster-Name> -Node ("<node1>","<node2>") –StaticAddress <n.n.n.n> -NoStorage
```   

# <a name="windows-server-2019"></a>[Windows Server 2019](#tab/windows2019)

Следующий сценарий PowerShell создает отказоустойчивый кластер. Обновите скрипт, введя имена узлов (имена виртуальных машин) и доступный IP-адрес из виртуальной сети Azure.

```powershell
New-Cluster -Name <FailoverCluster-Name> -Node ("<node1>","<node2>") –StaticAddress <n.n.n.n> -NoStorage -ManagementPointNetworkType Singleton 
```

Дополнительные сведения см. в статье [Отказоустойчивый кластер: сетевой объект кластера](https://blogs.windows.com/windowsexperience/2018/08/14/announcing-windows-server-2019-insider-preview-build-17733/#W0YAxO8BfwBRbkzG.97).

---

## <a name="configure-quorum"></a>Настройка кворума

Настройте решение кворума, которое лучше подходит для ваших бизнес-потребностей. Можно настроить диск- [свидетель](/windows-server/failover-clustering/manage-cluster-quorum#configure-the-cluster-quorum), [облачный следящий сервер](/windows-server/failover-clustering/deploy-cloud-witness)или [файловый ресурс-свидетель](/windows-server/failover-clustering/manage-cluster-quorum#configure-the-cluster-quorum). Дополнительные сведения см. [в статье кворум с виртуальными машинами SQL Server](hadr-cluster-best-practices.md#quorum). 

## <a name="validate-cluster"></a>Проверка кластера
Проверьте кластер в пользовательском интерфейсе или с помощью PowerShell.

Чтобы проверить кластер с помощью пользовательского интерфейса, выполните следующие действия на одной из виртуальных машин:

1. На панели **Диспетчер серверов** выберите **Инструменты** и нажмите **Диспетчер отказоустойчивости кластеров**.
1. В разделе **Диспетчер отказоустойчивости кластеров** выберите **Действие**, а затем выберите **Проверить конфигурацию**.
1. Выберите **Далее**.
1. На вкладке **Выбор серверов или кластера** введите имена обеих виртуальных машин.
1. На вкладке **Параметры тестирования** выберите **Выполнять только выбранные тесты**. 
1. Выберите **Далее**.
1. В разделе **Выбор теста** выберите все тесты, *Кроме* **хранилища** .

## <a name="test-cluster-failover"></a>Тестовая отработка отказа кластера

Протестируйте отработку отказа кластера. В **Диспетчер отказоустойчивости кластеров** щелкните правой кнопкой мыши кластер, выберите **другие действия**  >  **переместить основной кластер ресурс**  >  **выберите узел**, а затем выберите другой узел кластера. Перенесите основной кластерный ресурс на каждый узел кластера, а затем верните его на основной узел. Если вам удалось успешно перенести ресурс на каждый узел, вы готовы к установке SQL Server.  

:::image type="content" source="media/failover-cluster-instance-premium-file-share-manually-configure/test-cluster-failover.png" alt-text="Тестирование отказоустойчивости кластеров путем переноса основного кластерного ресурса на другие узлы":::

## <a name="create-sql-server-fci"></a>Создание экземпляра отказоустойчивого кластера SQL Server

После настройки отказоустойчивого кластера и всех компонентов кластера, включая хранилище, можно создать экземпляр отказоустойчивого кластера SQL Server.

1. Подключитесь к первой виртуальной машине с помощью протокол удаленного рабочего стола (RDP).

1. В **Диспетчер отказоустойчивости кластеров** убедитесь, что все основные ресурсы кластера находятся на первой виртуальной машине. При необходимости переместите все ресурсы на эту виртуальную машину.

1. Найдите установочный носитель. Если на виртуальной машине используется один из образов Azure Marketplace, носитель находится в папке `C:\SQLServer_<version number>_Full`. 

1. Выберите **Установить**.

1. В диалоговом окне **Центр установки SQL Server** выберите **Установка**.

1. Щелкните **Новая установка отказоустойчивого кластера SQL Server**. Следуйте указаниям мастера, чтобы установить экземпляр отказоустойчивого кластера SQL Server.

Каталоги данных FCI должны находиться на общих дисках Azure. 

1. После выполнения инструкций мастера программа установки установит экземпляр отказоустойчивого кластера SQL Server на первом узле.

1. После установки экземпляра отказоустойчивого кластера на первом узле подключитесь ко второму узлу с помощью протокола удаленного рабочего стола.

1. Откройте **центр установки SQL Server** и выберите пункт **Установка**.

1. Выберите **Добавление узла в отказоустойчивый кластер SQL Server**. Следуйте указаниям мастера, чтобы установить сервер SQL и добавить его в экземпляр отказоустойчивого кластера.

   >[!NOTE]
   >При использовании образа коллекции Azure Marketplace, который содержит SQL Server, средства SQL Server включаются в образ. Если вы не использовали один из этих образов, установите средства SQL Server отдельно. Дополнительные сведения см. в разделе [Скачивание SQL Server Management Studio (SSMS)](/sql/ssms/download-sql-server-management-studio-ssms).
   >

## <a name="register-with-the-sql-vm-rp"></a>Регистрация в RP виртуальной машины SQL

Чтобы управлять виртуальной машиной SQL Server на портале, зарегистрируйте ее с помощью расширения агента IaaS SQL (RP) в [режиме упрощенного управления](sql-agent-extension-manually-register-single-vm.md#lightweight-management-mode), в настоящее время единственным режимом, поддерживаемым FCI и SQL Server на виртуальных машинах Azure. 

Регистрация SQL Server виртуальной машины в упрощенном режиме с помощью PowerShell:  

```powershell-interactive
# Get the existing compute VM
$vm = Get-AzVM -Name <vm_name> -ResourceGroupName <resource_group_name>

# Register SQL VM with 'Lightweight' SQL IaaS agent
New-AzSqlVM -Name $vm.Name -ResourceGroupName $vm.ResourceGroupName -Location $vm.Location `
   -LicenseType PAYG -SqlManagementType LightWeight  
```

## <a name="configure-connectivity"></a>Настройка подключения 

Чтобы правильно направить трафик на текущий основной узел, настройте параметр подключения, подходящий для вашей среды. Вы можете создать [балансировщик нагрузки Azure](failover-cluster-instance-vnn-azure-load-balancer-configure.md) или, если вы используете SQL Server 2019 CU2 (или более поздней версии) и Windows Server 2016 (или более поздней версии), вместо этого можно использовать функцию [имени распределенной сети](failover-cluster-instance-distributed-network-name-dnn-configure.md) .  

Дополнительные сведения о вариантах подключения кластеров см. в статье [Рекомендации по настройке кластера (SQL Server на Виртуальных машинах Azure)](hadr-cluster-best-practices.md#connectivity). 

## <a name="limitations"></a>Ограничения

- Поддерживается только регистрация с расширением агента IaaS SQL в [режиме упрощенного управления](sql-server-iaas-agent-extension-automate-management.md#management-modes) .

## <a name="next-steps"></a>Дальнейшие действия

Если вы еще не сделали этого, настройте подключение к FCI с [именем виртуальной сети и подсистемой балансировки нагрузки Azure](failover-cluster-instance-vnn-azure-load-balancer-configure.md) или [именем распределенной сети (DNN)](failover-cluster-instance-distributed-network-name-dnn-configure.md). 

Если общие диски Azure не являются подходящим решением для хранения FCI, рассмотрите возможность создания FCI с помощью [файловых ресурсов Premium](failover-cluster-instance-premium-file-share-manually-configure.md) или [Локальные дисковые пространства](failover-cluster-instance-storage-spaces-direct-manually-configure.md) . 

Дополнительные сведения см. в обзоре [FCI с SQL Server на виртуальных машинах Azure и в](failover-cluster-instance-overview.md) разделе рекомендации по [конфигурации кластера](hadr-cluster-best-practices.md).

Дополнительные сведения см. в разделе: 
- [технологии кластера под управлением Windows](/windows-server/failover-clustering/failover-clustering-overview);   
- [Экземпляры отказоустойчивого кластера SQL Server](/sql/sql-server/failover-clusters/windows/always-on-failover-cluster-instances-sql-server)
