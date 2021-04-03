---
title: Сценарий PowerShell. Поиск хранилища для учетной записи хранения
description: Узнайте, как с помощью скрипта Azure PowerShell найти хранилище Служб восстановления, в котором зарегистрирована учетная запись хранения.
ms.topic: sample
ms.date: 1/28/2020
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 40859c1ea05210d27fcdcf33ba9d4f961965ea22
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "89075704"
---
# <a name="powershell-script-to-find-the-recovery-services-vault-where-a-storage-account-is-registered"></a>Скрипт PowerShell для поиска хранилища Служб восстановления, в котором зарегистрирована учетная запись хранения

Скрипт позволяет найти хранилище Служб восстановления, в котором зарегистрирована учетная запись хранения.

## <a name="sample-script"></a>Пример скрипта

```powershell
Param(
        [Parameter(Mandatory=$True)][System.String] $ResourceGroupName,
        [Parameter(Mandatory=$True)][System.String] $StorageAccountName,
        [Parameter(Mandatory=$True)][System.String] $SubscriptionId
    )

Connect-AzAccount
Select-AzSubscription -Subscription $SubscriptionId
$vaults = Get-AzRecoveryServicesVault
$found = $false
foreach($vault in $vaults)
{
  Write-Verbose "Checking vault: $($vault.Id)" -Verbose
  
  $containers = Get-AzRecoveryServicesBackupContainer -ContainerType AzureStorage -FriendlyName $StorageAccountName -ResourceGroupName $ResourceGroupName -VaultId $vault.Id -Status Registered
  
  if($containers -ne $null)
  {
    $found = $True
    Write-Information "Found Storage account $StorageAccountName registered in vault: $($vault.Id)" -InformationAction Continue
    break;
  }
}

if(!$found)
{
     Write-Information "Storage account: $StorageAccountName is not registered in any vault of this subscription" -InformationAction Continue
}
```

## <a name="how-to-execute-the-script"></a>Выполнение скрипта

1. Сохраните приведенный выше сценарий на компьютере, назвав его на свое усмотрение. В этом примере мы сохранили его как *FindRegisteredStorageAccount.ps1*.
2. Выполните скрипт, указав следующие параметры:

    * **-ResourceGroupName** — группа ресурсов учетной записи хранения;
    * **-StorageAccountName** — имя учетной записи хранения;
    * **-SubscriptionID** — идентификатор подписки, в которой содержится учетная запись хранения.

В следующем примере выполняется поиск хранилища Служб восстановления, в котором зарегистрирована учетная запись хранения *afsaccount*:

```powershell
.\FindRegisteredStorageAccount.ps1 -ResourceGroupName AzureFiles -StorageAccountName afsaccount -SubscriptionId ef4ad5a7-c2c0-4304-af80-af49f49af3d1
```

## <a name="output"></a>Выходные данные

В выходных данных отобразится полный путь к хранилищу Служб восстановления, в котором зарегистрирована учетная запись хранения. Пример выходных данных:

```output
Found Storage account afsaccount registered in vault: /subscriptions/ ef4ad5a7-c2c0-4304-af80-af49f49af3d1/resourceGroups/azurefiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault123
```

## <a name="next-steps"></a>Дальнейшие действия

См. сведения в [Back up Azure file shares in a Recovery Services vault](../backup-afs.md) (Резервное копирование Общих папок Azure из портала Azure)
