---
title: Обновление существующей политики резервного копирования виртуальных машин с помощью интерфейса командной строки
description: Узнайте, как обновить существующую политику резервного копирования виртуальных машин с помощью Azure CLI.
ms.topic: conceptual
ms.date: 12/31/2020
ms.openlocfilehash: 33083d6585d2b9296cd184ba258b8d2143d685b4
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98728584"
---
# <a name="update-the-existing-vm-backup-policy-using-cli"></a>Обновление существующей политики резервного копирования виртуальных машин с помощью интерфейса командной строки

Для обновления существующей политики резервного копирования виртуальных машин можно использовать Azure CLI. В этой статье объясняется, как экспортировать существующую политику в JSON-файл, изменить файл и затем использовать Azure CLI для обновления политики с измененной политикой.

## <a name="modify-an-existing-policy"></a>Изменение имеющейся политики

Чтобы изменить существующую политику резервного копирования виртуальной машины, выполните следующие действия.

1. Выполните команду [AZ Backup Policy показывать](/cli/azure/backup/policy#az_backup_policy_show) , чтобы получить сведения о политике, которую необходимо обновить.

    Пример

    ```azurecli
    az backup policy show --name testing123 --resource-group rg1234 --vault-name testvault
    ```

    В приведенном выше примере показаны сведения о политике виртуальной машины с именем *testing123*.

    Выходные данные:

    ```json
    {
    "eTag": null,
    "id": "/Subscriptions/efgsf-123-test-subscription/resourceGroups/rg1234/providers/Microsoft.RecoveryServices/vaults/testvault/backupPolicies/testing123",
    "location": null,
    "name": "testing123",
    "properties": {
        "backupManagementType": "AzureIaasVM",
        "instantRpDetails": {
        "azureBackupRgNamePrefix": null,
        "azureBackupRgNameSuffix": null
        },
        "instantRpRetentionRangeInDays": 2,
        "protectedItemsCount": 0,
        "retentionPolicy": {
        "dailySchedule": {
            "retentionDuration": {
            "count": 180,
            "durationType": "Days"
            },
            "retentionTimes": [
            "2020-08-03T04:30:00+00:00"
            ]
        },
        "monthlySchedule": null,
        "retentionPolicyType": "LongTermRetentionPolicy",
        "weeklySchedule": {
            "daysOfTheWeek": [
            "Sunday"
            ],
            "retentionDuration": {
            "count": 30,
            "durationType": "Weeks"
            },
            "retentionTimes": [
            "2020-08-03T04:30:00+00:00"
            ]
        },
        "yearlySchedule": null
        },
        "schedulePolicy": {
        "schedulePolicyType": "SimpleSchedulePolicy",
        "scheduleRunDays": null,
        "scheduleRunFrequency": "Daily",
        "scheduleRunTimes": [
            "2020-08-03T04:30:00+00:00"
        ],
        "scheduleWeeklyFrequency": 0
        },
        "timeZone": "UTC"
    },
    "resourceGroup": "azurefiles",
    "tags": null,
    "type": "Microsoft.RecoveryServices/vaults/backupPolicies"
    }
    ```

1. Сохраните приведенные выше выходные данные в JSON-файле. Например, сохраните его как *Policy.js*.
1. Обновите файл JSON в соответствии с вашими требованиями и сохраните изменения.

    Пример. чтобы обновить еженедельное хранение до 60 дней, обновите следующий раздел JSON файла, изменив значение счетчика на 60.

    ```json
            "retentionDuration": {
          "count": 60,
          "durationType": "Weeks"
        }

    ```

1. Сохраните изменения.
1. Выполните команду [AZ Backup Policy Set](/cli/azure/backup/policy#az_backup_policy_set) и передайте полный путь к обновленному файлу JSON в качестве значения параметра **--Policy** .

    ```azurecli
    az backup policy set --resource-group rg1234 --vault-name testvault --policy C:\temp2\Policy.json --name testing123
    ```

>[!NOTE]
>Вы также можете получить пример политики JSON, выполнив команду [AZ Backup Policy Get-Default-for-VM](/cli/azure/backup/policy#az_backup_policy_get_default_for_vm) .

## <a name="next-steps"></a>Дальнейшие действия

- [Управление резервными копиями виртуальных машин Azure с помощью службы Azure Backup](backup-azure-manage-vms.md)