---
title: Руководство по резервному копированию нескольких виртуальных машин Azure с помощью PowerShell
description: В этом учебнике рассматривается резервное копирование нескольких виртуальных машин Azure в хранилище Служб восстановления с помощью Azure PowerShell.
ms.topic: tutorial
ms.date: 03/05/2019
ms.custom: mvc, devx-track-azurepowershell
ms.openlocfilehash: 53a5df430a3ea57201ecb19bac68ef9f073beb14
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "90980973"
---
# <a name="back-up-azure-vms-with-powershell"></a>Резервное копирование виртуальных машин Azure с помощью PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Этот учебник содержит инструкции по развертыванию хранилища Служб восстановления [Azure Backup](backup-overview.md) для резервного копирования нескольких виртуальных машин Azure с помощью PowerShell.  

Из этого руководства вы узнаете, как выполнить следующие задачи:

> [!div class="checklist"]
>
> * Создание служб восстановления и указание контекста хранилища.
> * Определение политики архивации.
> * Применение политики архивации для защиты нескольких виртуальных машин.
> * Запустите задание резервного копирования по запросу для защищенных виртуальных машин. Прежде чем можно будет переходить к резервному копированию (или защите) виртуальной машины, вам потребуется выполнить [необходимые условия](backup-azure-arm-vms-prepare.md), чтобы подготовить среду для защиты виртуальных машин.

> [!IMPORTANT]
> В рамках руководства предполагается, что группа ресурсов и виртуальная машина Azure уже созданы.

## <a name="sign-in-and-register"></a>Вход и регистрация

1. Войдите в подписку Azure с помощью команды `Connect-AzAccount` и следуйте инструкциям на экране.

    ```powershell
    Connect-AzAccount
    ```

2. При первом использовании службы Azure Backup зарегистрируйте поставщик служб восстановления Azure в своей подписке с помощью команды [Register-AzResourceProvider](/powershell/module/az.Resources/Register-azResourceProvider). Если вы уже зарегистрировались, пропустите этот шаг.

    ```powershell
    Register-AzResourceProvider -ProviderNamespace "Microsoft.RecoveryServices"
    ```

## <a name="create-a-recovery-services-vault"></a>Создание хранилища Служб восстановления

[Хранилище служб восстановления](backup-azure-recovery-services-vault-overview.md) — это логический контейнер, в котором хранятся данные резервного копирования защищенных ресурсов, например виртуальных машин Azure. Когда выполняется задание резервного копирования, в хранилище Служб восстановления создается точка восстановления. Позже вы сможете использовать одну из этих точек восстановления, чтобы восстановить данные до определенной точки во времени.

* В этом учебнике создайте хранилище в той же группе ресурсов и расположении, в которых находится виртуальная машина для резервного копирования.
* Azure Backup автоматически обрабатывает хранилище для резервных копий данных. По умолчанию используется [геоизбыточное хранилище (GRS)](../storage/common/storage-redundancy.md#geo-redundant-storage). Геоизбыточность гарантирует, что данные резервного копирования реплицируются во вторичный регион, который находится в сотнях километров от основного.

Создайте хранилище следующим образом.

1. Чтобы создать хранилище, используйте командлет [New-AzRecoveryServicesVault](/powershell/module/az.recoveryservices/new-azrecoveryservicesvault). Укажите имя и расположение группы ресурсов виртуальной машины, для которой нужно создать резервную копию.

    ```powershell
    New-AzRecoveryServicesVault -Name myRSvault -ResourceGroupName "myResourceGroup" -Location "EastUS"
    ```

2. Для многих командлетов службы архивации Azure требуется объект хранилища служб восстановления в качестве входных данных. По этой причине объект хранилища Служб восстановления резервных копий удобно хранить в переменной.

    ```powershell
    $vault1 = Get-AzRecoveryServicesVault –Name myRSVault
    ```

3. Задайте контекст хранилища с помощью командлета [Set-AzRecoveryServicesVaultContext](/powershell/module/az.RecoveryServices/Set-azRecoveryServicesVaultContext).

   * Контекст хранилища — это тип данных, защищаемых в хранилище.
   * Заданный контекст применяется ко всем последующим командлетам.

     ```powershell
     Get-AzRecoveryServicesVault -Name "myRSVault" | Set-AzRecoveryServicesVaultContext
     ```

## <a name="back-up-azure-vms"></a>Резервное копирование виртуальных машин Azure

Резервные копии создаются в соответствии с расписанием, заданным в политике резервного копирования. Создаваемое хранилище служб восстановления поставляется с политиками защиты и хранения по умолчанию.

* Политика защиты по умолчанию запускает задание резервного копирования каждый день в указанное время.
* Политика хранения по умолчанию хранит ежедневную точку восстановления в течение 30 дней.

Чтобы включить и создавать резервные копии виртуальных машин Azure в рамках этого учебника, нужно сделать следующее:

1. Укажите контейнер в хранилище, которое содержит данные резервного копирования, с помощью командлета [Get-AzRecoveryServicesBackupContainer](/powershell/module/az.recoveryservices/get-Azrecoveryservicesbackupcontainer).
2. Каждая виртуальная машина для резервного копирования является элементом. Чтобы запустить задание резервного копирования, получите сведения о виртуальной машине с помощью командлета [Get-AzRecoveryServicesBackupItem](/powershell/module/az.recoveryservices/Get-AzRecoveryServicesBackupItem).
3. Выполните резервное копирование по требованию с помощью командлета [Backup-AzRecoveryServicesBackupItem](/powershell/module/az.recoveryservices/backup-Azrecoveryservicesbackupitem).
    * В ходе первого задания резервного копирования создается точка полного восстановления.
    * Во всех заданиях после начального резервного копирования создаются добавочные точки восстановления.
    * Добавочные точки восстановления требуют мало места и времени, так как они позволяют передать только изменения, внесенные с момента последнего резервного копирования.

Включите и запустите резервное копирование следующим образом:

```powershell
$namedContainer = Get-AzRecoveryServicesBackupContainer -ContainerType AzureVM -Status Registered -FriendlyName "V2VM"
$item = Get-AzRecoveryServicesBackupItem -Container $namedContainer -WorkloadType AzureVM
$job = Backup-AzRecoveryServicesBackupItem -Item $item
```

## <a name="troubleshooting"></a>Устранение неполадок

Если во время резервного копирования виртуальной машины возникнут проблемы, ознакомьтесь со [статьей об устранении неполадок](backup-azure-vms-troubleshoot.md).

### <a name="deleting-a-recovery-services-vault"></a>Удаление хранилища служб восстановления

Если требуется удалить хранилище, необходимо сначала удалить точки восстановления в хранилище, а затем отменить регистрацию хранилища следующим образом:

```powershell
$Cont = Get-AzRecoveryServicesBackupContainer -ContainerType AzureVM -Status Registered
$PI = Get-AzRecoveryServicesBackupItem -Container $Cont[0] -WorkloadType AzureVm
Disable-AzRecoveryServicesBackupProtection -RemoveRecoveryPoints $PI[0]
Unregister-AzRecoveryServicesBackupContainer -Container $namedContainer
Remove-AzRecoveryServicesVault -Vault $vault1
```

## <a name="next-steps"></a>Дальнейшие действия

* [Просмотрите](backup-azure-vms-automation.md) более подробное пошаговое руководство по резервному копированию и восстановлению виртуальных машин Azure с помощью PowerShell.
* [Manage Azure VM backups](backup-azure-manage-vms.md) (Управление резервными копиями виртуальных машин Azure)
* [Восстановление виртуальных машин Azure](backup-azure-arm-restore-vms.md)
