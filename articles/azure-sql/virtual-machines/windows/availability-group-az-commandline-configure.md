---
title: Настройка группы доступности (PowerShell & AZ CLI)
description: Для создания отказоустойчивого кластера Windows, прослушивателя группы доступности и внутренней подсистемы балансировки нагрузки на виртуальной машине SQL Server в Azure можно использовать либо PowerShell, либо Azure CLI.
services: virtual-machines-windows
documentationcenter: na
author: MashaMSFT
tags: azure-resource-manager
ms.service: virtual-machines-sql
ms.subservice: hadr
ms.topic: how-to
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 08/20/2020
ms.author: mathoma
ms.reviewer: jroth
ms.custom: seo-lt-2019, devx-track-azurecli
ms.openlocfilehash: 865ee3a5aeb8a2dd06d8759ba04d02259d2b4bee
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97359971"
---
# <a name="use-powershell-or-az-cli-to-configure-an-availability-group-for-sql-server-on-azure-vm"></a>Настройка группы доступности для SQL Server на виртуальной машине Azure с помощью PowerShell или AZ CLI 
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

В этой статье описывается, как использовать [PowerShell](/powershell/scripting/install/installing-powershell) или [Azure CLI](/cli/azure/sql/vm) для развертывания отказоустойчивого кластера Windows, добавления SQL Server виртуальных машин в кластер и создания внутренней подсистемы балансировки нагрузки и прослушивателя для Always on группы доступности. 

Развертывание группы доступности по-прежнему выполняется вручную с помощью SQL Server Management Studio (SSMS) или Transact-SQL (T-SQL). 

Хотя в этой статье используются PowerShell и AZ CLI для настройки среды группы доступности, ее также можно выполнить из [портал Azure](availability-group-azure-portal-configure.md), с помощью шаблонов быстрого запуска [Azure](availability-group-quickstart-template-configure.md)или [вручную](availability-group-manually-configure-tutorial.md) . 

## <a name="prerequisites"></a>Предварительные условия

Чтобы настроить Always On группы доступности, необходимо выполнить следующие предварительные требования. 

- [Подписка Azure](https://azure.microsoft.com/free/).
- Группа ресурсов с контроллером домена. 
- Одна или несколько виртуальных машин, присоединенных к домену, [в Azure под управлением SQL Server 2016 (или более поздней версии) Enterprise Edition](./create-sql-vm-portal.md) в *одной* группе доступности или *разных* зонах доступности, [зарегистрированных с расширением агента IaaS SQL](sql-agent-extension-manually-register-single-vm.md).  
- Последняя версия [PowerShell](/powershell/scripting/install/installing-powershell) или [Azure CLI](/cli/azure/install-azure-cli). 
- Два доступных (неиспользуемых сущностью) IP-адреса. Один предназначен для внутренней подсистемы балансировки нагрузки. Второй — для прослушивателя группы доступности в той же подсети, что и группа доступности. Если вы используете существующую подсистему балансировки нагрузки, для прослушивателя группы доступности требуется только один доступный IP-адрес. 

## <a name="permissions"></a>Разрешения

Для настройки группы доступности Always On с помощью Azure CLI необходимы разрешения следующих учетных записей: 

- Существующая учетная запись пользователя домена с разрешением на **создание объекта компьютера** в домене. Например, учетная запись администратора домена обычно имеет соответствующие разрешения (например: account@domain.com). _Эта учетная запись также должна входить в группу локальных администраторов на каждой виртуальной машине для создания кластера._
- Учетная запись пользователя домена, которая используется для управления SQL Server. 
 
## <a name="create-a-storage-account"></a>Создание учетной записи хранения 

Для кластера требуется учетная запись хранения, которая будет использоваться в качестве облака-свидетеля. Вы можете использовать существующую учетную запись хранения или создать ее. Если вы хотите использовать существующую учетную запись хранения, перейдите к следующему разделу. 

Следующий фрагмент кода создает учетную запись хранения: 

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli-interactive
# Create the storage account
# example: az storage account create -n 'cloudwitness' -g SQLVM-RG -l 'West US' `
#  --sku Standard_LRS --kind StorageV2 --access-tier Hot --https-only true

az storage account create -n <name> -g <resource group name> -l <region> `
  --sku Standard_LRS --kind StorageV2 --access-tier Hot --https-only true
```

>[!TIP]
> Если вы используете устаревшую версию Azure CLI, может появиться сообщение об ошибке `az sql: 'vm' is not in the 'az sql' command group`. Скачайте [последнюю версию Azure CLI](/cli/azure/install-azure-cli-windows), чтобы устранить эту ошибку.


# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```powershell-interactive
# Create the storage account
# example: New-AzStorageAccount -ResourceGroupName SQLVM-RG -Name cloudwitness `
#    -SkuName Standard_LRS -Location West US -Kind StorageV2 `
#    -AccessTier Hot -EnableHttpsTrafficOnly

New-AzStorageAccount -ResourceGroupName <resource group name> -Name <name> `
    -SkuName Standard_LRS -Location <region> -Kind StorageV2 `
    -AccessTier Hot -EnableHttpsTrafficOnly
```

---

## <a name="define-cluster-metadata"></a>Определение метаданных кластера

Группа команд [az sql vm group](/cli/azure/sql/vm/group) Azure CLI управляет метаданными службы отказоустойчивого кластера Windows Server (WSFC), в которой размещается группа доступности. К метаданным кластера относятся домен Active Directory, учетные записи кластера, учетные записи хранения, которые используются в роли облака-свидетеля, и версия SQL Server. Используйте [az sql vm group create](/cli/azure/sql/vm/group#az-sql-vm-group-create) для определения метаданных для WSFC, чтобы при добавлении первой виртуальной машины SQL Server был создан кластер в соответствии с определением. 

Следующий фрагмент кода определяет метаданные для кластера:

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli-interactive
# Define the cluster metadata
# example: az sql vm group create -n Cluster -l 'West US' -g SQLVM-RG `
#  --image-offer SQL2017-WS2016 --image-sku Enterprise --domain-fqdn domain.com `
#  --operator-acc vmadmin@domain.com --bootstrap-acc vmadmin@domain.com --service-acc sqlservice@domain.com `
#  --sa-key '4Z4/i1Dn8/bpbseyWX' `
#  --storage-account 'https://cloudwitness.blob.core.windows.net/'

az sql vm group create -n <cluster name> -l <region ex:eastus> -g <resource group name> `
  --image-offer <SQL2016-WS2016 or SQL2017-WS2016> --image-sku Enterprise --domain-fqdn <FQDN ex: domain.com> `
  --operator-acc <domain account ex: testop@domain.com> --bootstrap-acc <domain account ex:bootacc@domain.com> `
  --service-acc <service account ex: testservice@domain.com> `
  --sa-key '<PublicKey>' `
  --storage-account '<ex:https://cloudwitness.blob.core.windows.net/>'
```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```powershell-interactive
# Define the cluster metadata
# example: $group = New-AzSqlVMGroup -Name Cluster -Location West US ' 
#  -ResourceGroupName SQLVM-RG -Offer SQL2017-WS2016
#  -Sku Enterprise -DomainFqdn domain.com -ClusterOperatorAccount vmadmin@domain.com
#  -ClusterBootstrapAccount vmadmin@domain.com  -SqlServiceAccount sqlservice@domain.com 
#  -StorageAccountUrl '<ex:https://cloudwitness.blob.core.windows.net/>' `
#  -StorageAccountPrimaryKey '4Z4/i1Dn8/bpbseyWX'

$group = New-AzSqlVMGroup -Name <name> -Location <regio> 
  -ResourceGroupName <resource group name> -Offer <SQL201?-WS201?> 
  -Sku Enterprise -DomainFqdn <FQDN> -ClusterOperatorAccount <domain account> 
  -ClusterBootstrapAccount <domain account>  -SqlServiceAccount <service account> 
  -StorageAccountUrl '<ex:StorageAccountUrl>' `
  -StorageAccountPrimaryKey '<PublicKey>'
```

---

## <a name="add-vms-to-the-cluster"></a>Добавление виртуальных машин в кластер

При добавлении первой виртуальной машины SQL Server в кластер создается кластер. Вы можете создать кластер с указанным ранее именем, установить для него роль на виртуальных машинах SQL Server, а также добавить их в кластер с помощью команды [az sql vm add-to-group](/cli/azure/sql/vm#az-sql-vm-add-to-group). Выполните команду `az sql vm add-to-group` еще раз, чтобы добавить в созданный кластер дополнительные виртуальные машины SQL Server. 

Следующий фрагмент кода создает кластер и добавляет в него первую виртуальную машину SQL Server: 

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli-interactive
# Add SQL Server VMs to cluster
# example: az sql vm add-to-group -n SQLVM1 -g SQLVM-RG --sqlvm-group Cluster `
#  -b Str0ngAzur3P@ssword! -p Str0ngAzur3P@ssword! -s Str0ngAzur3P@ssword!
# example: az sql vm add-to-group -n SQLVM2 -g SQLVM-RG --sqlvm-group Cluster `
#  -b Str0ngAzur3P@ssword! -p Str0ngAzur3P@ssword! -s Str0ngAzur3P@ssword!

az sql vm add-to-group -n <VM1 Name> -g <Resource Group Name> --sqlvm-group <cluster name> `
  -b <bootstrap account password> -p <operator account password> -s <service account password>
az sql vm add-to-group -n <VM2 Name> -g <Resource Group Name> --sqlvm-group <cluster name> `
  -b <bootstrap account password> -p <operator account password> -s <service account password>
```
Используйте эту команду, чтобы добавить в кластер любые другие виртуальные машины SQL Server. Для имени виртуальной машины SQL Server измените только параметр `-n`. 

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```powershell-interactive
# Add SQL Server VMs to cluster
# example: $sqlvm1 = Get-AzSqlVM -Name SQLVM1 -ResourceGroupName SQLVM-RG
# $sqlvm2 = Get-AzSqlVM -Name SQLVM2  -ResourceGroupName SQLVM-RG

# $sqlvmconfig1 = Set-AzSqlVMConfigGroup -SqlVM $sqlvm1 `
#  -SqlVMGroup $group -ClusterOperatorAccountPassword Str0ngAzur3P@ssword! `
#  -SqlServiceAccountPassword Str0ngAzur3P@ssword! `
#  -ClusterBootstrapAccountPassword Str0ngAzur3P@ssword!

# $sqlvmconfig2 = Set-AzSqlVMConfigGroup -SqlVM $sqlvm2 `
#  -SqlVMGroup $group -ClusterOperatorAccountPassword Str0ngAzur3P@ssword! `
#  -SqlServiceAccountPassword Str0ngAzur3P@ssword! `
#  - ClusterBootstrapAccountPassword Str0ngAzur3P@ssword!

$sqlvm1 = Get-AzSqlVM -Name <VM1 Name> -ResourceGroupName <Resource Group Name>
$sqlvm2 = Get-AzSqlVM -Name <VM2 Name> -ResourceGroupName <Resource Group Name>

$sqlvmconfig1 = Set-AzSqlVMConfigGroup -SqlVM $sqlvm1 `
   -SqlVMGroup $group -ClusterOperatorAccountPassword <operator account password> `
   -SqlServiceAccountPassword <service account password> `
   -ClusterBootstrapAccountPassword <bootstrap account password>

Update-AzSqlVM -ResourceId $sqlvm1.ResourceId -SqlVM $sqlvmconfig1

$sqlvmconfig2 = Set-AzSqlVMConfigGroup -SqlVM $sqlvm2 `
   -SqlVMGroup $group -ClusterOperatorAccountPassword <operator account password> `
   -SqlServiceAccountPassword <service account password> `
   -ClusterBootstrapAccountPassword <bootstrap account password>

Update-AzSqlVM -ResourceId $sqlvm2.ResourceId -SqlVM $sqlvmconfig2
```

---


## <a name="validate-cluster"></a>Проверка кластера 

Чтобы отказоустойчивый кластер поддерживался корпорацией Майкрософт, он должен пройти проверку кластера. Подключитесь к виртуальной машине с помощью предпочтительного метода, например протокол удаленного рабочего стола (RDP), и убедитесь, что кластер прошел проверку, прежде чем продолжить. Несоблюдение этого действия оставляет кластер в неподдерживаемом состоянии. 

Вы можете проверить кластер с помощью диспетчер отказоустойчивости кластеров (FCM) или следующей команды PowerShell:

   ```powershell
   Test-Cluster –Node ("<node1>","<node2>") –Include "Inventory", "Network", "System Configuration"
   ```

## <a name="create-availability-group"></a>Создание группы доступности

Создайте группу доступности вручную, как обычно, с помощью [SQL Server Management Studio](/sql/database-engine/availability-groups/windows/use-the-availability-group-wizard-sql-server-management-studio), [PowerShell](/sql/database-engine/availability-groups/windows/create-an-availability-group-sql-server-powershell) или [Transact-SQL](/sql/database-engine/availability-groups/windows/create-an-availability-group-transact-sql). 

>[!IMPORTANT]
> *Не* создавайте прослушиватель на этом шаге. Это действие будет выполнено с помощью Azure CLI в следующих разделах.  

## <a name="create-internal-load-balancer"></a>Создание внутренней подсистемы балансировки нагрузки

[!INCLUDE [sql-ag-use-dnn-listener](../../includes/sql-ag-use-dnn-listener.md)]

Прослушивателю группы доступности Always On требуется внутренний экземпляр Azure Load Balancer. Внутренняя подсистема балансировки нагрузки предоставляет "плавающий" IP-адрес прослушивателю группы доступности, что обеспечивает более быструю отработку отказа и восстановление подключения. Если виртуальные машины SQL Server в группе доступности входят в одну группу доступности, вы можете использовать подсистему балансировки нагрузки ценовой категории "Базовый". В противном случае используйте подсистему балансировки нагрузки ценовой категории "Стандартный".  

> [!NOTE]
> Внутренняя подсистема балансировки нагрузки должна находиться в той же виртуальной сети, что и экземпляры виртуальных машин SQL Server. 

Следующий фрагмент кода создает внутреннюю подсистему балансировки нагрузки:

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli-interactive
# Create the internal load balancer
# example: az network lb create --name sqlILB -g SQLVM-RG --sku Standard `
# --vnet-name SQLVMvNet --subnet default

az network lb create --name sqlILB -g <resource group name> --sku Standard `
  --vnet-name <VNet Name> --subnet <subnet name>
```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```powershell-interactive
# Create the internal load balancer
# example: New-AzLoadBalancer -name sqlILB -ResourceGroupName SQLVM-RG  `
#  -Sku Standard -Location West US

New-AzLoadBalancer -name sqlILB -ResourceGroupName <resource group name> `
   -Sku Standard -Location <region>
```

---

>[!IMPORTANT]
> Ресурс общедоступного IP-адреса для каждой виртуальной машины SQL Server должен иметь номер SKU "Стандартный", чтобы обеспечить совместимость с подсистемой балансировки нагрузки ценовой категории "Стандартный". Чтобы определить номер SKU для ресурса общедоступного IP-адреса виртуальной машины, найдите **группу ресурсов** и выберите ресурс **Общедоступный IP-адрес** для требуемой виртуальной машины SQL Server, а затем найдите для него значение параметра **Номер SKU** на панели **Обзор**.  

## <a name="create-listener"></a>Создание прослушивателя

После создания группы доступности вручную можно создать прослушиватель с помощью команды [az sql vm ag-listener](/cli/azure/sql/vm/group/ag-listener#az-sql-vm-group-ag-listener-create). 

*Идентификатор ресурса подсети* — это значение `/subnets/<subnetname>`, которое добавляется к идентификатору ресурса виртуальной сети. Чтобы определить идентификатор ресурса подсети, выполните следующие действия:
   1. Перейдите к группе ресурсов на [портале Azure](https://portal.azure.com). 
   1. Выберите ресурс виртуальной сети. 
   1. Выберите **Свойства** в области **Параметры**. 
   1. Определите идентификатор ресурса для виртуальной сети и добавьте `/subnets/<subnetname>` к нему в конце, чтобы создать идентификатор ресурса подсети. Пример:
      - Ваш идентификатор ресурса виртуальной сети — `/subscriptions/a1a1-1a11a/resourceGroups/SQLVM-RG/providers/Microsoft.Network/virtualNetworks/SQLVMvNet`.
      - Ваше имя подсети — `default`.
      - Таким образом, ваш идентификатор ресурса подсети — `/subscriptions/a1a1-1a11a/resourceGroups/SQLVM-RG/providers/Microsoft.Network/virtualNetworks/SQLVMvNet/subnets/default`.


Следующий фрагмент кода создает прослушиватель группы доступности:

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli-interactive
# Create the availability group listener
# example: az sql vm group ag-listener create -n AGListener -g SQLVM-RG `
#  --ag-name SQLAG --group-name Cluster --ip-address 10.0.0.27 `
#  --load-balancer sqlilb --probe-port 59999  `
#  --subnet /subscriptions/a1a1-1a11a/resourceGroups/SQLVM-RG/providers/Microsoft.Network/virtualNetworks/SQLVMvNet/subnets/default `
#  --sqlvms sqlvm1 sqlvm2

az sql vm group ag-listener create -n <listener name> -g <resource group name> `
  --ag-name <availability group name> --group-name <cluster name> --ip-address <ag listener IP address> `
  --load-balancer <lbname> --probe-port <Load Balancer probe port, default 59999>  `
  --subnet <subnet resource id> `
  --sqlvms <names of SQL VM's hosting AG replicas, ex: sqlvm1 sqlvm2>
```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```powershell-interactive

# example: New-AzAvailabilityGroupListener -Name AGListener -ResourceGroupName SQLVM-RG `
#    -AvailabilityGroupName SQLAG  -GroupName Cluster `
#    -IpAddress 10.0.0.27 -LoadBalancerResourceId sqlilb `
#    -ProbePort 59999 `
#    -SubnetId /subscriptions/a1a1-1a11a/resourceGroups/SQLVM-RG/providers/Microsoft.Network/virtualNetworks/SQLVMvNet/subnets/default `
#    -SqlVirtualMachineId sqlvm1 sqlvm2


New-AzAvailabilityGroupListener -Name <listener name> -ResourceGroupName <resource group name> `
   -AvailabilityGroupName <availability group name> -GroupName <cluster name> `
   -IpAddress <ag listener IP address> -LoadBalancerResourceId <lbid> `
   -ProbePort <Load Balancer probe port, default 59999> `
   -SubnetId <subnet resource id> `
   -SqlVirtualMachineId <names of SQL VM's hosting AG replicas>
```

---

## <a name="modify-number-of-replicas"></a>Изменение числа реплик 
Существует дополнительный уровень сложности при развертывании группы доступности в виртуальных машинах SQL Server, размещенных в Azure. Поставщик ресурсов и группа виртуальных машин теперь управляют ресурсами. Таким образом, при добавлении или удалении реплик в группе доступности выполняется дополнительная задача добавления сведений о виртуальных машинах SQL Server в метаданные прослушивателя. При изменении числа реплик в группе доступности необходимо также использовать команду [az sql vm group ag-listener update](/cli/azure/sql/vm/group/ag-listener#az-sql-vm-group-ag-listener-update), чтобы добавить в прослушиватель метаданные виртуальных машин SQL Server. 


### <a name="add-a-replica"></a>Добавление реплики

Чтобы добавить новую реплику в группу доступности, выполните следующие действия:

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

1. Добавьте SQL Server виртуальную машину в группу кластера:
   ```azurecli-interactive

   # Add the SQL Server VM to the cluster group
   # example: az sql vm add-to-group -n SQLVM3 -g SQLVM-RG --sqlvm-group Cluster `
   # -b Str0ngAzur3P@ssword! -p Str0ngAzur3P@ssword! -s Str0ngAzur3P@ssword!

   az sql vm add-to-group -n <VM3 Name> -g <Resource Group Name> --sqlvm-group <cluster name> `
   -b <bootstrap account password> -p <operator account password> -s <service account password>
   ```

1. Используйте SQL Server Management Studio, чтобы добавить экземпляр SQL Server в качестве реплики в группу доступности.
1. Добавьте метаданные виртуальной машины SQL Server в прослушиватель, выполнив следующую команду:

   ```azurecli-interactive
   # Update the listener metadata with the new VM
   # example: az sql vm group ag-listener update -n AGListener `
   # -g sqlvm-rg --group-name Cluster --sqlvms sqlvm1 sqlvm2 sqlvm3

   az sql vm group ag-listener update -n <Listener> `
   -g <RG name> --group-name <cluster name> --sqlvms <SQL VMs, along with new SQL VM>
   ```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

1. Добавьте SQL Server виртуальную машину в группу кластера:

   ```powershell-interactive
   # Add the SQL Server VM to the cluster group
   # example: $sqlvm3 = Get-AzSqlVM -Name SQLVM3 -ResourceGroupName SQLVM-RG 
   # $group = Get-AzSqlVMGroup -ResourceGroupName SQLVM-RG -Name Cluster
   # $sqlvmconfig3 = Set-AzSqlVMConfigGroup -SqlVM $sqlvm3 -SqlVMGroup $group `
   #  -ClusterOperatorAccountPassword Str0ngAzur3P@ssword! `
   #  -SqlServiceAccountPassword Str0ngAzur3P@ssword! `
   #  -ClusterBootstrapAccountPassword Str0ngAzur3P@ssword!

   $sqlvm3 = Get-AzSqlVM -Name <VM1 Name> -ResourceGroupName <Resource Group Name>
   $group = Get-AzSqlVMGroup  -ResourceGroupName <resource group name> -Name <name>
   $sqlvmconfig3 = Set-AzSqlVMConfigGroup -SqlVM $sqlvm3 -SqlVMGroup $group `
   -ClusterOperatorAccountPassword <operator account password> `
   -SqlServiceAccountPassword <service account password> `
   -ClusterBootstrapAccountPassword <bootstrap account password>

   Update-AzSqlVM -ResourceId $sqlvm3.ResourceId -SqlVM $sqlvmconfig3
   ```

1. Используйте SQL Server Management Studio, чтобы добавить экземпляр SQL Server в качестве реплики в группу доступности.
1. Добавьте метаданные виртуальной машины SQL Server в прослушиватель, выполнив следующую команду:

   ```powershell-interactive
   # Update the listener metadata with the new VM
   # example: Update-AzAvailabilityGroupListener -Name AGListener -ResourceGroupName SQLVM-RG `
   #   -SqlVMGroupName Cluster -SqlVirtualMachineId SQLVM3

   Update-AzAvailabilityGroupListener -Name <Listener> -ResourceGroupName <Resource Group Name> `
      -SqlVMGroupName <cluster name> -SqlVirtualMachineId <SQL VMs, along with new SQL VM> 

   ```

---

### <a name="remove-a-replica"></a>Удаление реплики

Чтобы удалить реплику из группы доступности, выполните следующие действия:

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

1. Удалите реплику из группы доступности с помощью SQL Server Management Studio. 
1. Удалите метаданные виртуальной машины SQL Server из прослушивателя, выполнив следующую команду:
   ```azurecli-interactive
   # Update the listener metadata by removing the VM from the SQLVMs list
   # example: az sql vm group ag-listener update -n AGListener `
   # -g sqlvm-rg --group-name Cluster --sqlvms sqlvm1 sqlvm2

   az sql vm group ag-listener update -n <Listener> `
   -g <RG name> --group-name <cluster name> --sqlvms <SQL VMs that remain>
   ```
1. Удалите виртуальную машину SQL Server из кластера, выполнив следующую команду:
   ```azurecli-interactive
   # Remove the SQL VM from the cluster
   # example: az sql vm remove-from-group --name SQLVM3 --resource-group SQLVM-RG

   az sql vm remove-from-group --name <SQL VM name> --resource-group <RG name> 
   ```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

1. Удалите реплику из группы доступности с помощью SQL Server Management Studio. 
1. Удалите метаданные виртуальной машины SQL Server из прослушивателя, выполнив следующую команду:

   ```powershell-interactive
   # Update the listener metadata by removing the VM from the SQLVMs list
   # example: Update-AzAvailabilityGroupListener -Name AGListener -ResourceGroupName SQLVM-RG `
   #  -SqlVMGroupName Cluster -SqlVirtualMachineId SQLVM3

   Update-AzAvailabilityGroupListener -Name <Listener> -ResourceGroupName <Resource Group Name> `
      -SqlVMGroupName <cluster name> -SqlVirtualMachineId <SQL VMs that remain>

   ```
1. Удалите виртуальную машину SQL Server из кластера, выполнив следующую команду:

   ```powershell-interactive
   # Remove the SQL VM from the cluster
   # example: $sqlvm = Get-AzSqlVM -Name SQLVM3 -ResourceGroupName SQLVM-RG
   #  $sqlvm. SqlVirtualMachineGroup = ""
   #  Update-AzSqlVM -ResourceId $sqlvm -SqlVM $sqlvm

   $sqlvm = Get-AzSqlVM -Name <VM Name> -ResourceGroupName <Resource Group Name>
      $sqlvm. SqlVirtualMachineGroup = ""
    
    Update-AzSqlVM -ResourceId $sqlvm -SqlVM $sqlvm
   ```

---

## <a name="remove-listener"></a>Удаление прослушивателя
Если позже потребуется удалить прослушиватель группы доступности, настроенный с Azure CLI, необходимо пройти расширение агента IaaS SQL. Так как прослушиватель зарегистрирован с помощью расширения агента IaaS SQL, просто его удаление с помощью SQL Server Management Studio недостаточно. 

Лучший способ — удалить его через расширение агента IaaS SQL, используя следующий фрагмент кода в Azure CLI. Это приведет к удалению метаданных прослушивателя группы доступности из расширения агента IaaS SQL. а из группы доступности физически удаляется прослушиватель. 

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli-interactive
# Remove the availability group listener
# example: az sql vm group ag-listener delete --group-name Cluster --name AGListener --resource-group SQLVM-RG

az sql vm group ag-listener delete --group-name <cluster name> --name <listener name > --resource-group <resource group name>
```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```powershell-interactive
# Remove the availability group listener
# example: Remove-AzAvailabilityGroupListener -Name AGListener `
#   -ResourceGroupName SQLVM-RG -SqlVMGroupName Cluster

Remove-AzAvailabilityGroupListener -Name <Listener> `
   -ResourceGroupName <Resource Group Name> -SqlVMGroupName <cluster name>
```

---

## <a name="remove-cluster"></a>Удалить кластер

Удалите все узлы из кластера, чтобы уничтожить его, а затем удалите метаданные кластера из расширения агента IaaS SQL. Это можно сделать с помощью Azure CLI или PowerShell. 


# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Сначала удалите все SQL Server виртуальные машины из кластера: 

```azurecli-interactive
# Remove the VM from the cluster metadata
# example: az sql vm remove-from-group --name SQLVM2 --resource-group SQLVM-RG

az sql vm remove-from-group --name <VM1 name>  --resource-group <resource group name>
az sql vm remove-from-group --name <VM2 name>  --resource-group <resource group name>
```

Если это единственные виртуальные машины в кластере, кластер будет уничтожен. Если в кластере есть виртуальные машины, отличающиеся от SQL Server удаленных виртуальных машин, другие виртуальные машины не удаляются, и кластер не будет уничтожен. 

Затем удалите метаданные кластера из расширения агента IaaS SQL: 

```azurecli-interactive
# Remove the cluster from the SQL VM RP metadata
# example: az sql vm group delete --name Cluster --resource-group SQLVM-RG

az sql vm group delete --name <cluster name> Cluster --resource-group <resource group name>
```



# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Сначала удалите все SQL Server виртуальные машины из кластера. Это приведет к физическому удалению узлов из кластера и уничтожении кластера: 

```powershell-interactive
# Remove the SQL VM from the cluster
# example: $sqlvm = Get-AzSqlVM -Name SQLVM3 -ResourceGroupName SQLVM-RG
#  $sqlvm. SqlVirtualMachineGroup = ""
#  Update-AzSqlVM -ResourceId $sqlvm -SqlVM $sqlvm

$sqlvm = Get-AzSqlVM -Name <VM Name> -ResourceGroupName <Resource Group Name>
   $sqlvm. SqlVirtualMachineGroup = ""
   
   Update-AzSqlVM -ResourceId $sqlvm -SqlVM $sqlvm
```

Если это единственные виртуальные машины в кластере, кластер будет уничтожен. Если в кластере есть виртуальные машины, отличающиеся от SQL Server удаленных виртуальных машин, другие виртуальные машины не удаляются, и кластер не будет уничтожен. 

Затем удалите метаданные кластера из расширения агента IaaS SQL: 

```powershell-interactive
# Remove the cluster metadata
# example: Remove-AzSqlVMGroup -ResourceGroupName "SQLVM-RG" -Name "Cluster"

Remove-AzSqlVMGroup -ResourceGroupName "<resource group name>" -Name "<cluster name> "
```

---

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения см. в следующих статьях: 

* [Общие сведения о виртуальных машинах SQL Server](sql-server-on-azure-vm-iaas-what-is-overview.md)
* [Часто задаваемые вопросы о виртуальных машинах SQL Server](frequently-asked-questions-faq.md)
* [Заметки о виртуальных машинах SQL Server](../../database/doc-changes-updates-release-notes.md)
* [Изменение модели лицензирования для виртуальной машины SQL Server](licensing-model-azure-hybrid-benefit-ahb-change.md)
* [Общие сведения о группах доступности Always On (&#40;SQL Server&#41;)](/sql/database-engine/availability-groups/windows/overview-of-always-on-availability-groups-sql-server)   
* [Настройка экземпляра сервера для групп доступности Always On (&#40;SQL Server&#41;)](/sql/database-engine/availability-groups/windows/configuration-of-a-server-instance-for-always-on-availability-groups-sql-server)   
* [Администрирование группы доступности (&#40;SQL Server&#41;)](/sql/database-engine/availability-groups/windows/administration-of-an-availability-group-sql-server)   
* [Отслеживание групп доступности (SQL Server)](/sql/database-engine/availability-groups/windows/monitoring-of-availability-groups-sql-server)
* [Общие сведения об инструкциях Transact-SQL для групп доступности Always On (&#40;SQL Server&#41;)](/sql/database-engine/availability-groups/windows/transact-sql-statements-for-always-on-availability-groups)   
* [Обзор командлетов PowerShell для групп доступности Always On (&#40;SQL Server&#41;)](/sql/database-engine/availability-groups/windows/overview-of-powershell-cmdlets-for-always-on-availability-groups-sql-server)