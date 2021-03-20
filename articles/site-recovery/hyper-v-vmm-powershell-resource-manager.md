---
title: Настройка аварийного восстановления Hyper-V (с VMM) на дополнительный сайт с помощью Azure Site Recovery или PowerShell
description: В этой статье описана настройка аварийного восстановления виртуальных машин Hyper-V в облаках VMM на дополнительный сайт VMM с помощью Azure Site Recovery и PowerShell.
services: site-recovery
author: sujayt
manager: rochakm
ms.topic: article
ms.date: 1/10/2020
ms.author: sutalasi
ms.openlocfilehash: a4f7d330db9a4a0d9b435ebe7527e55e37c254e2
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "87086223"
---
# <a name="set-up-disaster-recovery-of-hyper-v-vms-to-a-secondary-site-by-using-powershell-resource-manager"></a>Настройка аварийного восстановления виртуальных машин Hyper-V на дополнительный сайт с помощью PowerShell (Resource Manager)

В этой статье показано, как автоматизировать шаги для репликации виртуальных машин Hyper-V из облаков System Center Virtual Machine Manager в облако Virtual Machine Manager на вторичном локальном сайте с помощью [Azure Site Recovery](site-recovery-overview.md).

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Предварительные требования

- Ознакомьтесь с [архитектурой и компонентами сценария](hyper-v-vmm-architecture.md).
- [Ознакомьтесь](./vmware-physical-secondary-support-matrix.md) с требованиями поддержки для всех компонентов.
- Убедитесь, что серверы Virtual Machine Manager и узлы Hyper-V соответствуют [требованиям поддержки](./vmware-physical-secondary-support-matrix.md).
- Убедитесь, что виртуальные машины, которые следует реплицировать, соответствуют [требованиям поддержки реплицируемых машин](./vmware-physical-secondary-support-matrix.md).

## <a name="prepare-for-network-mapping"></a>Подготовка к сопоставлению сетей

[Сопоставление сетей](hyper-v-vmm-network-mapping.md) происходит между локальными сетями виртуальных машин Virtual Machine Manager в исходном и целевом облаках. Это предоставляет следующие возможности:

- Подключение виртуальных машин к соответствующим целевым сетям виртуальных машин после отработки отказа.
- Оптимальное размещение виртуальных машин реплик на целевых серверах узлов Hyper-V.
- Если не настроить сетевое сопоставление, виртуальные машины реплик не будут подключены к каким-либо сетям виртуальных машин после отработки отказа.

Подготовьте Virtual Machine Manager следующим образом:

- Убедитесь, что на исходном и целевом серверах Virtual Machine Manager имеются [логические сети Virtual Machine Manager](/system-center/vmm/network-logical).
  - Логическая сеть на исходном сервере должна быть связана с исходным облаком, в котором находятся узлы Hyper-V.
  - Логическая сеть на целевом сервере должна быть связана с целевым облаком.
- Убедитесь, что на исходном и целевом серверах Virtual Machine Manager имеются [виртуальные сети виртуальной машины](/system-center/vmm/network-virtual). Сети виртуальных машин должны быть связаны с логической сетью в каждом расположении.
- Подключите виртуальные машины на исходных узлах Hyper-V к исходной сети виртуальных машин.

## <a name="prepare-for-powershell"></a>Подготовка для работы с PowerShell

Прежде чем приступать, убедитесь, что среда Azure PowerShell готова к работе.

- Если вы уже используете PowerShell, выполните обновление до версии 0.8.10 или более поздней. Дополнительные сведения о настройке PowerShell см. [здесь](/powershell/azure/).
- После установки и настройки PowerShell проверьте [командлеты службы](/powershell/azure/).
- Дополнительные сведения о том, как использовать значения параметров, входных и выходных данных в PowerShell, см. в руководстве [по началу работы](/powershell/azure/get-started-azureps).

## <a name="set-up-a-subscription"></a>Настройка подписки

1. Войдите в свою учетную запись Azure из PowerShell.

   ```azurepowershell
   $UserName = "<user@live.com>"
   $Password = "<password>"
   $SecurePassword = ConvertTo-SecureString -AsPlainText $Password -Force
   $Cred = New-Object System.Management.Automation.PSCredential -ArgumentList $UserName, $SecurePassword
   Connect-AzAccount #-Credential $Cred
   ```

1. Получите список подписок вместе с идентификаторами. Запишите идентификатор подписки, в которой необходимо создать хранилище служб восстановления.

   ```azurepowershell
   Get-AzSubscription
   ```

1. Настройте подписку для хранилища.

   ```azurepowershell
   Set-AzContext –SubscriptionID <subscriptionId>
   ```

## <a name="create-a-recovery-services-vault"></a>Создание хранилища Служб восстановления

1. Если у вас еще нет группы ресурсов Azure Resource Manager, создайте ее.

   ```azurepowershell
   New-AzResourceGroup -Name #ResourceGroupName -Location #location
   ```

1. Создайте хранилище служб восстановления. Сохраните объект хранилища в переменной, которая будет использоваться позже.

   ```azurepowershell
   $vault = New-AzRecoveryServicesVault -Name #vaultname -ResourceGroupName #ResourceGroupName -Location #location
   ```

   Объект хранилища можно получить после его создания с помощью `Get-AzRecoveryServicesVault` командлета.

## <a name="set-the-vault-context"></a>Задание контекста хранилища

1. Получите имеющееся хранилище.

   ```azurepowershell
   $vault = Get-AzRecoveryServicesVault -Name #vaultname
   ```

1. Задание контекста хранилища.

   ```azurepowershell
   Set-AzRecoveryServicesAsrVaultContext -Vault $vault
   ```

## <a name="install-the-site-recovery-provider"></a>Установка поставщика Site Recovery

1. На компьютере с Virtual Machine Manager создайте каталог, выполнив следующую команду:

   ```azurepowershell
   New-Item -Path C:\ASR -ItemType Directory
   ```

1. Извлеките файлы с помощью загруженного файла установки поставщика.

   ```console
   pushd C:\ASR\
   .\AzureSiteRecoveryProvider.exe /x:. /q
   ```

1. Начните установку поставщика и дождитесь ее завершения.

   ```console
   .\SetupDr.exe /i
   $installationRegPath = "HKLM:\Software\Microsoft\Microsoft System Center Virtual Machine Manager Server\DRAdapter"
   do
   {
     $isNotInstalled = $true;
     if(Test-Path $installationRegPath)
     {
       $isNotInstalled = $false;
     }
   }While($isNotInstalled)
   ```

1. Зарегистрируйте сервер в хранилище.

   ```console
   $BinPath = $env:SystemDrive+"\Program Files\Microsoft System Center 2012 R2\Virtual Machine Manager\bin"
   pushd $BinPath
   $encryptionFilePath = "C:\temp\".\DRConfigurator.exe /r /Credentials $VaultSettingFilePath /vmmfriendlyname $env:COMPUTERNAME /dataencryptionenabled $encryptionFilePath /startvmmservice
   ```

## <a name="create-and-associate-a-replication-policy"></a>Создание и связывание политики репликации

1. Создайте политику репликации (в данном случае для Hyper-V 2012 R2) следующим образом:

   ```azurepowershell
   $ReplicationFrequencyInSeconds = "300";        #options are 30,300,900
   $PolicyName = “replicapolicy”
   $RepProvider = HyperVReplica2012R2
   $Recoverypoints = 24                    #specify the number of hours to retain recovery points
   $AppConsistentSnapshotFrequency = 4 #specify the frequency (in hours) at which app consistent snapshots are taken
   $AuthMode = "Kerberos"  #options are "Kerberos" or "Certificate"
   $AuthPort = "8083"  #specify the port number that will be used for replication traffic on Hyper-V hosts
   $InitialRepMethod = "Online" #options are "Online" or "Offline"

   $policyresult = New-AzRecoveryServicesAsrPolicy -Name $policyname -ReplicationProvider $RepProvider -ReplicationFrequencyInSeconds $Replicationfrequencyinseconds -NumberOfRecoveryPointsToRetain $recoverypoints -ApplicationConsistentSnapshotFrequencyInHours $AppConsistentSnapshotFrequency -Authentication $AuthMode -ReplicationPort $AuthPort -ReplicationMethod $InitialRepMethod
   ```

   > [!NOTE]
   > Облако Virtual Machine Manager может содержать узлы Hyper-V под управлением разных версий Windows Server, но политика репликации зависит от версии ОС. Если вы используете узлы на разных версиях операционных систем, создайте отдельные политики репликации для каждой системы. Например, если у вас есть пять узлов, работающих с Windows Server 2012, и три — с Windows Server 2012 R2, создайте две политики репликации. По одной для каждого типа ОС.

1. Получите первичный контейнер защиты (основного облака Virtual Machine Manager) и контейнер защиты для восстановления (облако Virtual Machine Manager восстановления).

   ```azurepowershell
   $PrimaryCloud = "testprimarycloud"
   $primaryprotectionContainer = Get-AzRecoveryServicesAsrProtectionContainer -FriendlyName $PrimaryCloud;

   $RecoveryCloud = "testrecoverycloud"
   $recoveryprotectionContainer = Get-AzRecoveryServicesAsrProtectionContainer -FriendlyName $RecoveryCloud;
   ```

1. Извлеките созданную политику репликации, используя понятное имя.

   ```azurepowershell
   $policy = Get-AzRecoveryServicesAsrPolicy -FriendlyName $policyname
   ```

1. Запустите связывание контейнера защиты (облака Virtual Machine Manager) с политикой репликации.

   ```azurepowershell
   $associationJob  = New-AzRecoveryServicesAsrProtectionContainerMapping -Policy $Policy -PrimaryProtectionContainer $primaryprotectionContainer -RecoveryProtectionContainer $recoveryprotectionContainer
   ```

1. Дождитесь завершения задания связывания политики. Используйте приведенный ниже фрагмент кода PowerShell, чтобы проверить, завершено ли задание.

   ```azurepowershell
   $job = Get-AzRecoveryServicesAsrJob -Job $associationJob

   if($job -eq $null -or $job.StateDescription -ne "Completed")
   {
     $isJobLeftForProcessing = $true;
   }
   ```

1. После того, как задание завершило обработку, выполните следующую команду:

   ```azurepowershell
   if($isJobLeftForProcessing)
   {
     Start-Sleep -Seconds 60
   }
   While($isJobLeftForProcessing)
   ```

Для проверки выполнения операции выполните действия, описанные в разделе [Мониторинг действий](#monitor-activity).

##  <a name="configure-network-mapping"></a>Настройка сетевого сопоставления

1. Используйте эту команду, чтобы получить серверы для текущего хранилища. Команда сохраняет Site Recovery серверы в `$Servers` переменной массива.

   ```azurepowershell
   $Servers = Get-AzRecoveryServicesAsrFabric
   ```

1. Чтобы извлечь сети для исходного и целевого серверов Virtual Machine Manager, выполните следующую команду:

   ```azurepowershell
   $PrimaryNetworks = Get-AzRecoveryServicesAsrNetwork -Fabric $Servers[0]

   $RecoveryNetworks = Get-AzRecoveryServicesAsrNetwork -Fabric $Servers[1]
   ```

   > [!NOTE]
   > Исходный сервер Virtual Machine Manager может быть первым или вторым в массиве серверов. Проверьте имена серверов Virtual Machine Manager и получите сети соответствующим образом.

1. Этот командлет создает сопоставление между основной сетью и сетью восстановления. Он указывает основную сеть в качестве первого элемента `$PrimaryNetworks` . В нем указывается сеть восстановления в качестве первого элемента `$RecoveryNetworks` .

   ```azurepowershell
   New-AzRecoveryServicesAsrNetworkMapping -PrimaryNetwork $PrimaryNetworks[0] -RecoveryNetwork $RecoveryNetworks[0]
   ```

## <a name="enable-protection-for-vms"></a>Включение защиты для виртуальных машин

После успешной настройки серверов, облаков и сетей можно включить защиту для виртуальных машин в облаке.

1. Чтобы включить защиту, выполните следующую команду для получения контейнера защиты.

   ```azurepowershell
   $PrimaryProtectionContainer = Get-AzRecoveryServicesAsrProtectionContainer -FriendlyName $PrimaryCloudName
   ```

1. Получите сущность защиты (виртуальной машины) следующим образом:

   ```azurepowershell
   $protectionEntity = Get-AzRecoveryServicesAsrProtectableItem -FriendlyName $VMName -ProtectionContainer $PrimaryProtectionContainer
   ```

1. Включите репликацию для виртуальной машины.

   ```azurepowershell
   $jobResult = New-AzRecoveryServicesAsrReplicationProtectedItem -ProtectableItem $protectionentity -ProtectionContainerMapping $policy -VmmToVmm
   ```

> [!NOTE]
> Если вы хотите выполнить репликацию на управляемые диски с поддержкой CMK в Azure, выполните следующие действия с помощью команды AZ PowerShell 3.3.0/назад:
>
> 1. Включение отработки отказа на управляемые диски путем обновления свойств виртуальной машины
> 1. Используйте `Get-AzRecoveryServicesAsrReplicationProtectedItem` командлет, чтобы получить идентификатор диска для каждого диска защищенного элемента.
> 1. Создайте объект Dictionary с помощью `New-Object "System.Collections.Generic.Dictionary``2[System.String,System.String]"` командлета, который будет содержать сопоставление идентификатора диска с набором шифрования диска. Эти наборы шифрования дисков должны быть предварительно созданы в целевом регионе.
> 1. Обновите свойства виртуальной машины с помощью `Set-AzRecoveryServicesAsrReplicationProtectedItem` командлета, передав объект Dictionary в параметре **дискидтодискенкриптионсетмап** .

## <a name="run-a-test-failover"></a>Запуск тестовой отработки отказа

Чтобы протестировать развертывание, запустите тестовую отработку отказа для одной виртуальной машины. Также можно создать план восстановления, который содержит несколько виртуальных машин, и запустить для него тестовую отработку отказа. Тестовая обработка отказа имитирует механизм отработки отказа и восстановления в изолированной сети.

1. Получите виртуальную машину, куда будет выполнена отработка отказа виртуальных машин.

   ```azurepowershell
   $Servers = Get-AzRecoveryServicesASRFabric
   $RecoveryNetworks = Get-AzRecoveryServicesAsrNetwork -Name $Servers[1]
   ```

1. Выполните тестовую отработка отказа.

   Для одной виртуальной машины:

   ```azurepowershell
   $protectionEntity = Get-AzRecoveryServicesAsrProtectableItem -FriendlyName $VMName -ProtectionContainer $PrimaryprotectionContainer

   $jobIDResult = Start-AzRecoveryServicesAsrTestFailoverJob -Direction PrimaryToRecovery -ReplicationProtectedItem $protectionEntity -VMNetwork $RecoveryNetworks[1]
   ```

   Для плана восстановления:

   ```azurepowershell
   $recoveryplanname = "test-recovery-plan"

   $recoveryplan = Get-AzRecoveryServicesAsrRecoveryPlan -FriendlyName $recoveryplanname

   $jobIDResult = Start-AzRecoveryServicesAsrTestFailoverJob -Direction PrimaryToRecovery -RecoveryPlan $recoveryplan -VMNetwork $RecoveryNetworks[1]
   ```

Для проверки выполнения операции выполните действия, описанные в разделе [Мониторинг действий](#monitor-activity).

## <a name="run-planned-and-unplanned-failovers"></a>Запуск плановых и внеплановых отработок отказа

1. Выполните плановую отработку отказа.

   Для одной виртуальной машины:

   ```azurepowershell
   $protectionEntity = Get-AzRecoveryServicesAsrProtectableItem -Name $VMName -ProtectionContainer $PrimaryprotectionContainer

   $jobIDResult = Start-AzRecoveryServicesAsrPlannedFailoverJob -Direction PrimaryToRecovery -ReplicationProtectedItem $protectionEntity
   ```

   Для плана восстановления:

   ```azurepowershell
   $recoveryplanname = "test-recovery-plan"

   $recoveryplan = Get-AzRecoveryServicesAsrRecoveryPlan -FriendlyName $recoveryplanname

   $jobIDResult = Start-AzRecoveryServicesAsrPlannedFailoverJob -Direction PrimaryToRecovery -RecoveryPlan $recoveryplan
   ```

1. Выполните внеплановую отработку отказа.

   Для одной виртуальной машины:

   ```azurepowershell
   $protectionEntity = Get-AzRecoveryServicesAsrProtectableItem -Name $VMName -ProtectionContainer $PrimaryprotectionContainer

   $jobIDResult = Start-AzRecoveryServicesAsrUnplannedFailoverJob -Direction PrimaryToRecovery -ReplicationProtectedItem $protectionEntity
   ```

   Для плана восстановления:

   ```azurepowershell
   $recoveryplanname = "test-recovery-plan"

   $recoveryplan = Get-AzRecoveryServicesAsrRecoveryPlan -FriendlyName $recoveryplanname

   $jobIDResult = Start-AzRecoveryServicesAsrUnplannedFailoverJob -Direction PrimaryToRecovery -RecoveryPlan $recoveryplan
   ```

## <a name="monitor-activity"></a>Мониторинг действий

Используйте следующие команды для мониторинга действий отработки отказа. Дождитесь завершения обработки, прежде чем запускать следующие задания.

```azurepowershell
Do
{
    $job = Get-AzRecoveryServicesAsrJob -TargetObjectId $associationJob.JobId;
    Write-Host "Job State:{0}, StateDescription:{1}" -f Job.State, $job.StateDescription;
    if($job -eq $null -or $job.StateDescription -ne "Completed")
    {
        $isJobLeftForProcessing = $true;
    }

if($isJobLeftForProcessing)
    {
        Start-Sleep -Seconds 60
    }
}While($isJobLeftForProcessing)
```

## <a name="next-steps"></a>Дальнейшие действия

[Узнайте больше](/powershell/module/az.recoveryservices) об использовании командлетов PowerShell Resource Manager для службы Site Recovery.
