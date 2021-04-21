---
title: Руководство по Управление резервными копиями баз данных SAP HANA с помощью интерфейса командной строки
description: Из этого учебника вы узнаете, как управлять резервными копиями баз данных SAP HANA, запущенных на виртуальной машине Azure, через интерфейс командной строки Azure.
ms.topic: tutorial
ms.date: 12/4/2019
ms.custom: devx-track-azurecli
ms.openlocfilehash: 7090701e3642fd9703737060e0876c8bbfc27994
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107765185"
---
# <a name="tutorial-manage-sap-hana-databases-in-an-azure-vm-using-azure-cli"></a>Руководство по Управление базами данных SAP HANA на виртуальных машинах Azure с помощью Azure CLI

Azure CLI используется для создания ресурсов Azure и управления ими из командной строки или с помощью скриптов. В этой документации содержатся сведения об управлении сохраненной резервной копией базы данных SAP HANA на виртуальной машине Azure с помощью Azure CLI. Эти действия можно также выполнить с помощью [портала Azure](./sap-hana-db-manage.md).

Вы можете выполнять команды интерфейса командной строки с помощью [Azure Cloud Shell](tutorial-sap-hana-backup-cli.md).

Изучив это руководство, вы сможете выполнить следующие действия:

> [!div class="checklist"]
>
> * выполнить мониторинг заданий резервного копирования и восстановления;
> * обеспечить защиту новых баз данных, добавленных в экземпляр SAP HANA;
> * изменить политику;
> * Остановить защиту
> * Возобновить защиту

Если для настройки резервного копирования базы данных SAP HANA вы следовали инструкциям из [руководства по резервному копированию баз данных SAP HANA в Azure c помощью CLI](tutorial-sap-hana-backup-cli.md), вы используете следующие ресурсы:

* группа ресурсов с именем *saphanaResourceGroup*;
* хранилище с именем *saphanaVault*;
* защищенный контейнер с именем *VMAppContainer;Compute;saphanaResourceGroup;saphanaVM*;
* резервная копия базы данных или элемента с именем *saphanadatabase;hxe;hxe*;
* ресурсы в регионе *westus2*.

Интерфейс командной строки Azure позволяет легко управлять базой данных SAP HANA на виртуальной машине Azure, для которой выполняется резервное копирование с помощью Azure Backup. В этом учебнике подробно описаны все операции управления.

## <a name="monitor-backup-and-restore-jobs"></a>выполнить мониторинг заданий резервного копирования и восстановления;

Для наблюдения за завершенными или выполняющимися в данный момент заданиями (резервного копирования или восстановления) используйте командлет [az backup job list](/cli/azure/backup/job#az_backup_job_list). CLI также позволяет [приостановить выполняющееся задание](/cli/azure/backup/job#az_backup_job_stop) или [подождать, пока задание не завершится](/cli/azure/backup/job#az_backup_job_wait).

```azurecli-interactive
az backup job list --resource-group saphanaResourceGroup \
    --vault-name saphanaVault \
    --output table
```

Результат должен выглядеть следующим образом.

```output
Name                                  Operation              Status      Item Name       Start Time UTC
------------------------------------  ---------------        ---------   ----------      -------------------  
e0f15dae-7cac-4475-a833-f52c50e5b6c3  ConfigureBackup        Completed   hxe             2019-12-03T03:09:210831+00:00  
ccdb4dce-8b15-47c5-8c46-b0985352238f  Backup (Full)          Completed   hxe [hxehost]   2019-12-01T10:30:58.867489+00:00
4980af91-1090-49a6-ab96-13bc905a5282  Backup (Differential)  Completed   hxe [hxehost]   2019-12-01T10:36:00.563909+00:00
F7c68818-039f-4a0f-8d73-e0747e68a813  Restore (Log)          Completed   hxe [hxehost]   2019-12-03T05:44:51.081607+00:00
```

## <a name="change-policy"></a>Изменение политики

Чтобы изменить политику, лежащую в основе конфигурации резервного копирования SAP HANA, воспользуйтесь командлетом [az backup policy set](/cli/azure/backup/policy#az_backup_policy_set). Параметр name в этом командлете ссылается на элемент архивации, политику которого нужно изменить. В этом руководстве мы заменим политику *saphanadatabase;hxe;hxe* для базы данных SAP HANA новой политикой *newsaphanaPolicy*. Новые политики можно создавать с помощью командлета [az backup policy create](/cli/azure/backup/policy#az_backup_policy_create).

```azurecli-interactive
az backup item set policy --resource-group saphanaResourceGroup \
    --vault-name saphanaVault \
    --container-name VMAppContainer;Compute;saphanaResourceGroup;saphanaVM \
    --policy-name newsaphanaPolicy \
    --name saphanadatabase;hxe;hxe \
```

Выходные данные должны выглядеть так:

```output
Name                                  Resource Group
------------------------------------- --------------
cb110094-9b15-4c55-ad45-6899200eb8dd  SAPHANA
```

## <a name="create-incremental-backup-policy"></a>Создание политики добавочного резервного копирования

Чтобы создать политику добавочного резервного копирования, выполните команду [az backup policy create](/cli/azure/backup/policy#az_backup_policy_create) со следующими параметрами:

* **--backup-management-type** — рабочая нагрузка Azure.
* **--workload-type** — SAPHana.
* **--name** — имя политики.
* **--policy** — файл JSON с соответствующими сведениями о расписании и хранении.
* **--resource-group** — группа ресурсов хранилища.
* **--vault-name** — имя хранилища.

Пример:

```azurecli
az backup policy create --resource-group saphanaResourceGroup --vault-name saphanaVault --name sappolicy --backup-management-type AzureWorkload --policy sappolicy.json --workload-type SAPHana
```

Пример JSON (sappolicy.json):

```json
  "eTag": null,
  "id": "/Subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/saphanaResourceGroup/providers/Microsoft.RecoveryServices/vaults/saphanaVault/backupPolicies/sappolicy",
  "location": null,
  "name": "sappolicy",
  "properties": {
    "backupManagementType": "AzureWorkload",
    "makePolicyConsistent": null,
    "protectedItemsCount": 0,
    "settings": {
      "isCompression": false,
      "issqlcompression": false,
      "timeZone": "UTC"
    },
    "subProtectionPolicy": [
      {
        "policyType": "Full",
        "retentionPolicy": {
          "dailySchedule": null,
          "monthlySchedule": {
            "retentionDuration": {
              "count": 60,
              "durationType": "Months"
            },
            "retentionScheduleDaily": null,
            "retentionScheduleFormatType": "Weekly",
            "retentionScheduleWeekly": {
              "daysOfTheWeek": [
                "Sunday"
              ],
              "weeksOfTheMonth": [
                "First"
              ]
            },
            "retentionTimes": [
              "2021-01-19T00:30:00+00:00"
            ]
          },
          "retentionPolicyType": "LongTermRetentionPolicy",
          "weeklySchedule": {
            "daysOfTheWeek": [
              "Sunday"
            ],
            "retentionDuration": {
              "count": 104,
              "durationType": "Weeks"
            },
            "retentionTimes": [
              "2021-01-19T00:30:00+00:00"
            ]
          },
          "yearlySchedule": {
            "monthsOfYear": [
              "January"
            ],
            "retentionDuration": {
              "count": 10,
              "durationType": "Years"
            },
            "retentionScheduleDaily": null,
            "retentionScheduleFormatType": "Weekly",
            "retentionScheduleWeekly": {
              "daysOfTheWeek": [
                "Sunday"
              ],
              "weeksOfTheMonth": [
                "First"
              ]
            },
            "retentionTimes": [
              "2021-01-19T00:30:00+00:00"
            ]
          }
        },
        "schedulePolicy": {
          "schedulePolicyType": "SimpleSchedulePolicy",
          "scheduleRunDays": [
            "Sunday"
          ],
          "scheduleRunFrequency": "Weekly",
          "scheduleRunTimes": [
            "2021-01-19T00:30:00+00:00"
          ],
          "scheduleWeeklyFrequency": 0
        }
      },
      {
        "policyType": "Incremental",
        "retentionPolicy": {
          "retentionDuration": {
            "count": 30,
            "durationType": "Days"
          },
          "retentionPolicyType": "SimpleRetentionPolicy"
        },
        "schedulePolicy": {
          "schedulePolicyType": "SimpleSchedulePolicy",
          "scheduleRunDays": [
            "Monday",
            "Tuesday",
            "Wednesday",
            "Thursday",
            "Friday",
            "Saturday"
          ],
          "scheduleRunFrequency": "Weekly",
          "scheduleRunTimes": [
            "2017-03-07T02:00:00+00:00"
          ],
          "scheduleWeeklyFrequency": 0
        }
      },
      {
        "policyType": "Log",
        "retentionPolicy": {
          "retentionDuration": {
            "count": 15,
            "durationType": "Days"
          },
          "retentionPolicyType": "SimpleRetentionPolicy"
        },
        "schedulePolicy": {
          "scheduleFrequencyInMins": 120,
          "schedulePolicyType": "LogSchedulePolicy"
        }
      }
    ],
    "workLoadType": "SAPHanaDatabase"
  },
  "resourceGroup": "saphanaResourceGroup",
  "tags": null,
  "type": "Microsoft.RecoveryServices/vaults/backupPolicies"
} 
```

После успешного создания политики в выходных данных команды будет выведен код JSON политики, которую вы передали в качестве параметра при выполнении команды.

Вы можете изменить следующий раздел политики, чтобы указать требуемую периодичность резервного копирования и срок хранения для добавочных резервных копий.

Пример:

```json
{
  "policyType": "Incremental",
  "retentionPolicy": {
    "retentionDuration": {
      "count": 30,
      "durationType": "Days"
    },
    "retentionPolicyType": "SimpleRetentionPolicy"
  },
  "schedulePolicy": {
    "schedulePolicyType": "SimpleSchedulePolicy",
    "scheduleRunDays": [
      "Monday",
      "Tuesday",
      "Wednesday",
      "Thursday",
      "Friday",
      "Saturday"
    ],
    "scheduleRunFrequency": "Weekly",
    "scheduleRunTimes": [
      "2017-03-07T02:00:00+00:00"
    ],
    "scheduleWeeklyFrequency": 0
  }
}
```

Пример:

Если требуется выполнять добавочное резервное копирование только в субботу и хранить копии в течение 60 дней, внесите следующие изменения в политику:

* Измените значение **retentionDuration** на 60 дней.
* Укажите для **ScheduleRunDays** только субботу.

```json
 {
  "policyType": "Incremental",
  "retentionPolicy": {
    "retentionDuration": {
      "count": 60,
      "durationType": "Days"
    },
    "retentionPolicyType": "SimpleRetentionPolicy"
  },
  "schedulePolicy": {
    "schedulePolicyType": "SimpleSchedulePolicy",
    "scheduleRunDays": [
      "Saturday"
    ],
    "scheduleRunFrequency": "Weekly",
    "scheduleRunTimes": [
      "2017-03-07T02:00:00+00:00"
    ],
    "scheduleWeeklyFrequency": 0
  }
}
```

## <a name="protect-new-databases-added-to-an-sap-hana-instance"></a>обеспечить защиту новых баз данных, добавленных в экземпляр SAP HANA;

[При регистрации экземпляра SAP HANA в хранилище Служб восстановления](tutorial-sap-hana-backup-cli.md#register-and-protect-the-sap-hana-instance) автоматически обнаруживаются все базы данных в этом экземпляре.

Но в том случае, если новые базы данных добавляются в экземпляр SAP HANA позднее, следует использовать командлет [az backup protectable-item initialize](/cli/azure/backup/protectable-item#az_backup_protectable_item_initialize). Этот командлет обнаруживает вновь добавленные базы данных.

```azurecli-interactive
az backup protectable-item initialize --resource-group saphanaResourceGroup \
    --vault-name saphanaVault \
    --container-name VMAppContainer;Compute;saphanaResourceGroup;saphanaVM \
    --workload-type SAPHANA
```

После этого выполните командлет [az backup protectable-item list](/cli/azure/backup/protectable-item#az_backup_protectable_item_list), чтобы получить список всех баз данных, обнаруженных в экземпляре SAP HANA. Но этот список не включает базы данных, для которых уже настроено резервное копирование. После обнаружения базы данных, для которой требуется резервное копирование, воспользуйтесь [учебником по включению резервного копирования для базы данных SAP HANA](tutorial-sap-hana-backup-cli.md#enable-backup-on-sap-hana-database).

```azurecli-interactive
az backup protectable-item list --resource-group saphanaResourceGroup \
    --vault-name saphanaVault \
    --workload-type SAPHANA \
    --output table
```

В списке появится новая база данных, для которой вы решили включить резервное копирование, и теперь он будет выглядеть следующим образом:

```output
Name                            Protectable Item Type    ParentName    ServerName    IsProtected
---------------------------     ----------------------   ------------  -----------   ------------
saphanasystem;hxe               SAPHanaSystem            HXE           hxehost       NotProtected  
saphanadatabase;hxe;systemdb    SAPHanaDatabase          HXE           hxehost       NotProtected
saphanadatabase;hxe;newhxe      SAPHanaDatabase          HXE           hxehost       NotProtected
```

## <a name="stop-protection-for-an-sap-hana-database"></a>Остановка защиты для базы данных SAP HANA

Вы можете остановить создание резервных копий базы данных SQL Server несколькими способами:

* можно остановить все будущие задания резервного копирования и удалить все точки восстановления;
* остановить все будущие задания резервного копирования, но сохранить точки восстановления без изменений.

Если вы решили сохранить точки восстановления, учитывайте следующие факторы:

* все точки восстановления навсегда остаются неизменными, и любая очистка прекращается одновременно с отключением защиты;
* вы будете оплачивать работу защищенного экземпляр и используемый объем хранилища;
* если вы удалите источник данных без остановки резервного копирования, новые операции резервного копирования будут завершаться сбоем.

Давайте немного подробнее рассмотрим каждый из способов отключения защиты.

### <a name="stop-protection-with-retain-data"></a>Отключите защиту с сохранением данных

Чтобы отключить защиту с сохранением данных, используйте командлет [az backup protection disable](/cli/azure/backup/protection#az_backup_protection_disable).

```azurecli-interactive
az backup protection disable --resource-group saphanaResourceGroup \
    --vault-name saphanaVault \
    --container-name VMAppContainer;Compute;saphanaResourceGroup;saphanaVM \
    --item-name saphanadatabase;hxe;hxe \
    --workload-type SAPHANA \
    --output table
```

Выходные данные должны выглядеть так:

```output
Name                                  ResourceGroup
------------------------------------  ---------------  
g0f15dae-7cac-4475-d833-f52c50e5b6c3  saphanaResourceGroup
```

Чтобы проверить состояние этой операции, используйте командлет [az backup job show](/cli/azure/backup/job#az_backup_job_show).

### <a name="stop-protection-without-retain-data"></a>Отключение защиты без сохранения данных

Чтобы отключить защиту без сохранения данных, используйте командлет [az backup protection disable](/cli/azure/backup/protection#az_backup_protection_disable).

```azurecli-interactive
az backup protection disable --resource-group saphanaResourceGroup \
    --vault-name saphanaVault \
    --container-name VMAppContainer;Compute;saphanaResourceGroup;saphanaVM \
    --item-name saphanadatabase;hxe;hxe \
    --workload-type SAPHANA \
    --delete-backup-data true \
    --output table
```

Выходные данные должны выглядеть так:

```output
Name                                  ResourceGroup
------------------------------------  ---------------  
g0f15dae-7cac-4475-d833-f52c50e5b6c3  saphanaResourceGroup
```

Чтобы проверить состояние этой операции, используйте командлет [az backup job show](/cli/azure/backup/job#az_backup_job_show).

## <a name="resume-protection"></a>Возобновить защиту

Если вы отключите защиту для базы данных SAP HANA с сохранением данных, позднее можно будет возобновить защиту. Если вы не сохраните существующие данные, защиту возобновить не удастся.

Чтобы возобновить защиту, используйте командлет [az backup protection resume](/cli/azure/backup/protection#az_backup_protection_resume).

```azurecli-interactive
az backup protection resume --resource-group saphanaResourceGroup \
    --vault-name saphanaVault \
    --container-name VMAppContainer;Compute;saphanaResourceGroup;saphanaVM \
    --policy-name saphanaPolicy \
    --output table
```

Выходные данные должны выглядеть так:

```output
Name                                  ResourceGroup
------------------------------------  ---------------  
b2a7f108-1020-4529-870f-6c4c43e2bb9e  saphanaResourceGroup
```

Чтобы проверить состояние этой операции, используйте командлет [az backup job show](/cli/azure/backup/job#az_backup_job_show).

## <a name="next-steps"></a>Дальнейшие действия

* Сведения о резервном копировании базы данных SAP HANA, работающей на виртуальной машине Azure, с помощью портала Azure см. в статье [о резервном копировании баз данных SAP HANA на виртуальных машинах Azure](./backup-azure-sap-hana-database.md).

* Сведения об управлении резервными копиями базы данных SAP HANA, работающей на виртуальной машине Azure, с помощью портала Azure см. в статье [об управлении резервными копиями баз данных SAP HANA на виртуальных машинах Azure](./sap-hana-db-manage.md).