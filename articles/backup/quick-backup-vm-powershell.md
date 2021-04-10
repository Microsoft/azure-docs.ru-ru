---
title: Краткое руководство. Резервное копирование виртуальной машины с помощью PowerShell
description: В этом кратком руководстве вы узнаете, как выполнять резервное копирование виртуальных машин Azure с помощью модуля Azure PowerShell.
ms.devlang: azurecli
ms.topic: quickstart
ms.date: 04/16/2019
ms.custom: mvc, devx-track-azurepowershell
ms.openlocfilehash: 81005a3032355f566aef2a6794cf2ec5038b0dd8
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98737122"
---
# <a name="back-up-a-virtual-machine-in-azure-with-powershell"></a>Резервное копирование виртуальной машины в Azure с помощью PowerShell

Модуль [Azure PowerShell AZ](/powershell/azure/new-azureps-module-az) используется для создания и администрирования ресурсов Azure с помощью командной строки или скриптов.

[Azure Backup](backup-overview.md) создает резервные копии локальных компьютеров, приложений и виртуальных машин Azure. В этой статье показано, как создать резервную копию виртуальной машины Azure с помощью модуля AZ. Кроме того, вы можете выполнить резервное копирование виртуальной машины с помощью [Azure CLI](quick-backup-vm-cli.md) или на [портале Azure](quick-backup-vm-portal.md).

В этом руководстве объясняется, как включить резервное копирование существующей виртуальной машины Azure. Если вам необходимо создать виртуальную машину, см. руководство по [созданию виртуальной машины с помощью Azure PowerShell](/previous-versions/azure/virtual-machines/scripts/virtual-machines-windows-powershell-sample-create-vm?toc=%2fpowershell%2fmodule%2ftoc.json).

Для работы с этим кратким руководством требуется модуль Azure PowerShell AZ 1.0.0 или более поздней версии. Чтобы узнать версию, выполните команду `Get-Module -ListAvailable Az`. Если вам необходимо выполнить установку или обновление, см. статью [об установке модуля Azure PowerShell](/powershell/azure/install-az-ps).

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="sign-in-and-register"></a>Вход и регистрация

1. Войдите в подписку Azure с помощью команды `Connect-AzAccount` и следуйте инструкциям на экране.

    ```powershell
    Connect-AzAccount
    ```

2. При первом использовании службы Azure Backup зарегистрируйте поставщик служб восстановления Azure в своей подписке с помощью команды [Register-AzResourceProvider](/powershell/module/az.Resources/Register-azResourceProvider) следующим образом:

    ```powershell
    Register-AzResourceProvider -ProviderNamespace "Microsoft.RecoveryServices"
    ```

## <a name="create-a-recovery-services-vault"></a>Создание хранилища служб восстановления

[Хранилище служб восстановления](backup-azure-recovery-services-vault-overview.md) — это логический контейнер, в котором хранятся данные резервного копирования защищенных ресурсов, например виртуальных машин Azure. Когда выполняется задание резервного копирования, в хранилище Служб восстановления создается точка восстановления. Позже вы сможете использовать одну из этих точек восстановления, чтобы восстановить данные до определенной точки во времени.

При создании хранилища:

- Укажите расположение и группу ресурсов виртуальной машины, для которой необходимо создать резервную копию.
- Если вы создали виртуальную машину с помощью этого [примера скрипта](/previous-versions/azure/virtual-machines/scripts/virtual-machines-windows-powershell-sample-create-vm?toc=%2fpowershell%2fmodule%2ftoc.json), группой ресурсов будет **myResourceGroup**, виртуальной машиной — **_myVM_*, а ресурсы будут находиться в регионе **Западная Европа**.
- Azure Backup автоматически обрабатывает хранилище для резервных копий данных. По умолчанию используется [геоизбыточное хранилище (GRS)](../storage/common/storage-redundancy.md#geo-redundant-storage). Геоизбыточность гарантирует, что данные резервного копирования реплицируются во вторичный регион, который находится в сотнях километров от основного.

Теперь создайте хранилище:

1. Чтобы создать хранилище, используйте командлет [New-AzRecoveryServicesVault](/powershell/module/az.recoveryservices/new-azrecoveryservicesvault):

    ```powershell
    New-AzRecoveryServicesVault `
        -ResourceGroupName "myResourceGroup" `
        -Name "myRecoveryServicesVault" `
    -Location "WestEurope"
    ```

2. Задайте контекст хранилища с помощью [Set-AzRecoveryServicesVaultContext](/powershell/module/az.RecoveryServices/Set-azRecoveryServicesVaultContext) следующим образом:

    ```powershell
    Get-AzRecoveryServicesVault `
        -Name "myRecoveryServicesVault" | Set-AzRecoveryServicesVaultContext
    ```

3. Измените конфигурацию избыточности хранилища (LRS/GRS) с помощью командлета [Set-AzRecoveryServicesBackupProperty](/powershell/module/az.recoveryservices/set-azrecoveryservicesbackupproperty) следующим образом.

    ```powershell
    Get-AzRecoveryServicesVault `
        -Name "myRecoveryServicesVault" | Set-AzRecoveryServicesBackupProperty -BackupStorageRedundancy LocallyRedundant/GeoRedundant
    ```

    > [!NOTE]
    > Избыточность хранилища можно изменить только в том случае, если в хранилище нет защищенных резервных копий.

## <a name="enable-backup-for-an-azure-vm"></a>Включение резервного копирования для виртуальной машины Azure

Включите резервное копирование для виртуальной машины Azure и укажите политику резервного копирования.

- Политика определяет время запуска резервного копирования и время хранения точек восстановления, созданных при этой операции.
- Согласно политике защиты по умолчанию резервное копирование выполняется каждый день, а точки восстановления хранятся в течение 30 дней. Вы можете использовать политику по умолчанию, чтобы быстро защитить виртуальную машину.

Включите резервное копирование следующим образом:

1. Сначала задайте политику по умолчанию с помощью командлета [Get-AzRecoveryServicesBackupProtectionPolicy](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupprotectionpolicy):

    ```powershell
    $policy = Get-AzRecoveryServicesBackupProtectionPolicy     -Name "DefaultPolicy"
    ```

2. Включите резервное копирование виртуальной машины с помощью [Enable-AzRecoveryServicesBackupProtection](/powershell/module/az.recoveryservices/enable-azrecoveryservicesbackupprotection). Укажите политику, группу ресурсов и имя виртуальной машины.

    ```powershell
    Enable-AzRecoveryServicesBackupProtection `
        -ResourceGroupName "myResourceGroup" `
        -Name "myVM" `
        -Policy $policy
    ```

## <a name="start-a-backup-job"></a>Запуск задания резервного копирования

Резервные копии выполняются в соответствии с расписанием, заданным в политике резервного копирования. Вы также можете выполнить резервное копирование по запросу:

- В ходе первого задания резервного копирования создается точка полного восстановления.
- Во всех заданиях после начального резервного копирования создаются добавочные точки восстановления.
- Добавочные точки восстановления требуют мало места и времени, так как они позволяют передать только изменения, внесенные с момента последнего резервного копирования.

Чтобы выполнить резервное копирование по запросу, используйте командлет [Backup-AzRecoveryServicesBackupItem](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupitem).

- Укажите контейнер в хранилище, которое содержит данные резервного копирования, с помощью командлета [Get-AzRecoveryServicesBackupContainer](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupcontainer).
- Каждая виртуальная машина для резервного копирования обрабатывается как элемент. Чтобы запустить задание резервного копирования, получите сведения о виртуальной машине с помощью командлета [Get-AzRecoveryServicesBackupItem](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupitem).

Чтобы выполнить резервное копирование по запросу, сделайте следующее:

1. Укажите контейнер, получите сведения о виртуальной машине и запустите резервное копирование.

    ```powershell
    $backupcontainer = Get-AzRecoveryServicesBackupContainer `
        -ContainerType "AzureVM" `
        -FriendlyName "myVM"

    $item = Get-AzRecoveryServicesBackupItem `
        -Container $backupcontainer `
        -WorkloadType "AzureVM"

    Backup-AzRecoveryServicesBackupItem -Item $item
    ```

2. Процесс может занять до 20 минут, так как в ходе первого задания резервного копирования создается точка полного восстановления. Отслеживайте ход выполнения задания, как описано в следующей процедуре.

## <a name="monitor-the-backup-job"></a>Мониторинг задания резервного копирования

1. Выполните [Get-AzRecoveryservicesBackupJob](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupjob) для проверки состояния задания.

    ```powershell
    Get-AzRecoveryservicesBackupJob
    ```

    В результате вы получите приблизительно следующие выходные данные. Они указывают на то, что задание **выполняется**:

    ```output
    WorkloadName   Operation         Status       StartTime              EndTime                JobID
    ------------   ---------         ------       ---------              -------                -----
    myvm           Backup            InProgress   9/18/2017 9:38:02 PM                          9f9e8f14
    myvm           ConfigureBackup   Completed    9/18/2017 9:33:18 PM   9/18/2017 9:33:51 PM   fe79c739
    ```

2. Когда задание находится в состоянии **Завершено**, виртуальная машина защищена и для нее создана точка полного восстановления.

## <a name="clean-up-the-deployment"></a>Очистка развертывания

Если вам больше не требуется резервная копия виртуальной машины, ее можно удалить.

- Если вы хотите попытаться восстановить виртуальною машину, пропустите удаление.
- Если вы использовали имеющуюся виртуальную машину, можете пропустить последний командлет [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup), чтобы оставить группу ресурсов и виртуальную машину.

Отключите защиту, удалите точки восстановления и хранилище. Затем удалите группу ресурсов и связанные ресурсы виртуальной машины следующим образом:

```powershell
Disable-AzRecoveryServicesBackupProtection -Item $item -RemoveRecoveryPoints
$vault = Get-AzRecoveryServicesVault -Name "myRecoveryServicesVault"
Remove-AzRecoveryServicesVault -Vault $vault
Remove-AzResourceGroup -Name "myResourceGroup"
```

## <a name="next-steps"></a>Дальнейшие действия

В этом кратком руководстве вы создали хранилище служб восстановления, включили защиту на виртуальной машине и создали начальную точку восстановления.

- [Узнайте, как](tutorial-backup-vm-at-scale.md) создать резервные копии виртуальных машин на портале Azure.
- [Узнайте, как](tutorial-restore-disk.md) быстро восстановить виртуальную машину.