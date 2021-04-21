---
title: Руководство по восстановлению базы данных SAP HANA в Azure с помощью интерфейса командной строки (CLI)
description: Из этого руководства вы узнаете, как восстанавливать базы данных SAP HANA, запущенные на виртуальной машине Azure, из хранилища Служб восстановления для Azure Backup с помощью Azure CLI.
ms.topic: tutorial
ms.date: 12/4/2019
ms.custom: devx-track-azurecli
ms.openlocfilehash: a249ab63aa72c1d39ab1626e72ff3b2037f3f723
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107768457"
---
# <a name="tutorial-restore-sap-hana-databases-in-an-azure-vm-using-azure-cli"></a>Руководство по Восстановление баз данных SAP HANA на виртуальных машинах Azure с помощью Azure CLI

Azure CLI используется для создания ресурсов Azure и управления ими из командной строки или с помощью скриптов. В этой документации содержатся сведения о восстановлении базы данных SAP HANA из резервной копии на виртуальной машине Azure с помощью Azure CLI. Эти действия можно также выполнить с помощью [портала Azure](./sap-hana-db-restore.md).

Вы можете выполнять команды интерфейса командной строки с помощью [Azure Cloud Shell](tutorial-sap-hana-backup-cli.md).

Изучив это руководство, вы сможете выполнить следующие действия:

> [!div class="checklist"]
>
> * Просмотр точек восстановления для базы данных с резервным копированием.
> * Восстановление базы данных

В этом руководстве предполагается, что у вас есть база данных SAP HANA на виртуальной машине Azure, для которой выполняется резервное копирование с помощью Azure Backup. Если для настройки резервного копирования базы данных SAP HANA вы следовали инструкциям из [руководства по резервному копированию баз данных SAP HANA в Azure c помощью CLI](tutorial-sap-hana-backup-cli.md), вы используете следующие ресурсы:

* группа ресурсов с именем *saphanaResourceGroup*;
* хранилище с именем *saphanaVault*;
* защищенный контейнер с именем *VMAppContainer;Compute;saphanaResourceGroup;saphanaVM*;
* резервная копия базы данных или элемента с именем *saphanadatabase;hxe;hxe*;
* ресурсы в регионе *westus2*.

## <a name="view-restore-points-for-a-backed-up-database"></a>Просмотр точек восстановления для базы данных с резервным копированием

Чтобы просмотреть полный список существующих точек восстановления для базы данных, используйте командлет [az backup recoverypoint list](/cli/azure/backup/recoverypoint#az_backup_recoverypoint_show_log_chain), как показано ниже.

```azurecli-interactive
az backup recoverypoint list --resource-group saphanaResourceGroup \
    --vault-name saphanaVault \
    --container-name VMAppContainer;Compute;saphanaResourceGroup;saphanaVM \
    --item-name saphanadatabase;hxe;hxe \
   --output table
```

Список точек восстановления будет выглядеть следующим образом.

```output
Name                      Time                               BackupManagementType   Item Name               RecoveryPointType
-------------------       ---------------------------------  ---------------------  ----------------------  ------------------
7660777527047692711       2019-12-10T04:00:32.346000+00:00   AzureWorkload          SAPHanaDtabase;hxe;hxe  Full
7896624824685666836       2019-12-15T10:33:32.346000+00:00   AzureWorkload          SAPHanaDtabase;hxe;hxe  Differential
DefaultRangeRecoveryPoint                                    AzureWorkload          SAPHanaDtabase;hxe;hxe  Log
```

В приведенном выше списке вы видите три точки восстановления, по одной для полной копии, разностной копии и резервной копии журналов.

>[!NOTE]
>Также с помощью командлета [az backup recoverypoint show-log-chain](/cli/azure/backup/recoverypoint#az_backup_recoverypoint_show_log_chain) можно просмотреть начальную и конечную точки для каждой полной цепочки резервных копий журналов.

## <a name="prerequisites-to-restore-a-database"></a>Предварительные условия для восстановления базы данных

Перед восстановлением базы данных убедитесь, что выполняются следующие условия.

* Базу данных можно восстановить только на экземпляре SAP HANA, расположенном в том же регионе.
* Целевой экземпляр должен быть зарегистрирован в том же хранилище, что и исходный сервер.
* Azure Backup не может обнаруживать два разных экземпляра SAP HANA на одной виртуальной машине. Это означает, что восстановление данных из одного экземпляра в другой экземпляр на той же виртуальной машине невозможно.

## <a name="restore-a-database"></a>Восстановление базы данных

В Azure Backup можно выполнить восстановление баз данных SAP HANA, запущенных на виртуальных машинах Azure, следующим образом.

* Восстановление до состояния на определенную дату или время (с точностью до секунд) с помощью резервных копий журналов. На основе выбранного времени Azure Backup автоматически определяет соответствующие полные или разностные резервные копии и цепочку резервных копий журналов, необходимых для восстановления.
* Восстановление конкретной полной или разностной резервной копии по определенной точке восстановления.

Чтобы восстановить базу данных, используйте командлет [az restore restore-azurewl](/cli/azure/backup/restore#az_backup_restore_restore_azurewl), который принимает в качестве входных данных объект конфигурации восстановления. Этот объект можно создать с помощью командлета [az backup recoveryconfig show](/cli/azure/backup/recoveryconfig#az_backup_recoveryconfig_show). Объект конфигурации восстановления содержит все сведения для выполнения восстановления. Среди прочего он определяет режим восстановления: **OriginalWorkloadRestore** или **AlternateWorkloadRestore**.

>[!NOTE]
> **OriginalWorkloadRestore** обозначает восстановление данных в том же экземпляре SAP HANA, где находилась исходная база данных-источник. В этом варианте перезаписывается исходная база данных. <br>
> **AlternateWorkloadRestore** обозначает восстановление базы данных в альтернативное расположение с сохранением исходной базы данных-источника.

## <a name="restore-to-alternate-location"></a>Восстановление в альтернативном расположении

Чтобы восстановить базу данных в альтернативном расположении, используйте режим восстановления **AlternateWorkloadRestore**. Для него нужно выбрать точку восстановления, например последнюю из существующих или любую другую.

В рамках этого руководства восстановление выполняется до последней точки восстановления. [Просмотрите список точек восстановления](#view-restore-points-for-a-backed-up-database) для базы данных и выберите наиболее подходящую. В рамках этого руководства используется точка восстановления с именем *7660777527047692711*.

Используя указанные выше имя точки восстановления и режим восстановления, создайте объект конфигурации восстановления, используя командлет [az backup recoveryconfig show](/cli/azure/backup/recoveryconfig#az_backup_recoveryconfig_show). Давайте узнаем, что означают остальные параметры в этом командлете.

* **--target-item-name** содержит имя, которое будет использоваться для восстанавливаемой базы данных. В нашем примере используется имя *restored_database*.
* **--target-server-name** обозначает имя сервера SAP HANA, который зарегистрирован в хранилище Служб восстановления и размещается в том же регионе, что и восстанавливаемая база данных. В рамках этого руководства мы восстановим базу данных на том же защищенном сервере SAP HANA с именем *hxehost*.
* **--target-server-type** должен иметь значение **HANAInstance** для восстановления баз данных SAP HANA.

```azurecli-interactive

az backup recoveryconfig show --resource-group saphanaResourceGroup \
    --vault-name saphanaVault \
    --container-name VMAppContainer;Compute;saphanaResourceGroup;saphanaVM \
    --item-name saphanadatabase;hxe;hxe \
    --restore-mode AlternateWorkloadRestore \
    --rp-name 7660777527047692711 \
    --target-item-name restored_database \
    --target-server-name hxehost \
    --target-server-type HANAInstance \
    --workload-type SAPHANA \
    --output json
```

В ответ на приведенный выше запрос вы получите примерно такой объект конфигурации восстановления.

```output
{"restore_mode": "AlternateLocation", "container_uri": " VMAppContainer;Compute;saphanaResourceGroup;saphanaVM ", "item_uri": "SAPHanaDatabase;hxe;hxe", "recovery_point_id": "7660777527047692711", "item_type": "SAPHana", "source_resource_id": "/subscriptions/ef4ab5a7-c2c0-4304-af80-af49f48af3d1/resourceGroups/saphanaResourceGroup/providers/Microsoft.Compute/virtualMachines/saphanavm", "database_name": null, "container_id": null, "alternate_directory_paths": null}
```

Теперь выполните командлет [az restore restore-azurewl](/cli/azure/backup/restore#az_backup_restore_restore_azurewl) для восстановления базы данных. Чтобы использовать эту команду, мы предоставим приведенный выше код в формате JSON, сохраненный в файл с именем *recoveryconfig.json*.

```azurecli-interactive
az backup restore restore-azurewl --resource-group saphanaResourceGroup \
    --vault-name saphanaVault \
    --restore-config recoveryconfig.json \
    --output table
```

Выходные данные будут выглядеть следующим образом:

```output
Name                                  Resource
------------------------------------  -------
5b198508-9712-43df-844b-977e5dfc30ea  SAPHANA
```

В ответе вы получите имя задания. Это имя задания позволяет отслеживать состояние задания с помощью [az backup job show](/cli/azure/backup/job#az_backup_job_show).

## <a name="restore-and-overwrite"></a>Восстановление и перезапись

Чтобы восстановить базу данных в исходном расположении, мы укажем режим восстановления **OrignialWorkloadRestore**. Для него нужно выбрать точку восстановления, например последнюю из существующих или любую другую.

В этом руководстве мы выберем последнюю точку восстановления на момент времени "28-11-2019-09:53:00". Эту точку восстановления можно указать в следующих форматах: дд-мм-гггг, дд-мм-гггг-чч:мм:сс. Чтобы выбрать допустимый момент времени для восстановления, используйте командлет [az backup recoverypoint show-log-chain](/cli/azure/backup/recoverypoint#az_backup_recoverypoint_show_log_chain), который позволяет получить интервалы полных цепочек резервных копий журналов.

```azurecli-interactive
az backup recoveryconfig show --resource-group saphanaResourceGroup \
    --vault-name saphanaVault \
    --container-name VMAppContainer;Compute;saphanaResourceGroup;saphanaVM \
    --item-name saphanadatabase;hxe;hxe \
    --restore-mode OriginalWorkloadRestore \
    --log-point-in-time 28-11-2019-09:53:00 \
    --output json
```

В ответ на приведенный выше запрос вы получите примерно такой объект конфигурации восстановления.

```output
{"restore_mode": "OriginalLocation", "container_uri": " VMAppContainer;Compute;saphanaResourceGroup;saphanaVM ", "item_uri": "SAPHanaDatabase;hxe;hxe", "recovery_point_id": "DefaultRangeRecoveryPoint", "log_point_in_time": "28-11-2019-09:53:00", "item_type": "SAPHana", "source_resource_id": "/subscriptions/ef4ab5a7-c2c0-4304-af80-af49f48af3d1/resourceGroups/saphanaResourceGroup/providers/Microsoft.Compute/virtualMachines/saphanavm", "database_name": null, "container_id": null, "alternate_directory_paths": null}"
```

Теперь выполните командлет [az restore restore-azurewl](/cli/azure/backup/restore#az_backup_restore_restore_azurewl) для восстановления базы данных. Чтобы использовать эту команду, мы предоставим приведенный выше код в формате JSON, сохраненный в файл с именем *recoveryconfig.json*.

```azurecli-interactive
az backup restore restore-azurewl --resource-group saphanaResourceGroup \
    --vault-name saphanaVault \
    --restore-config recoveryconfig.json \
    --output table
```

Выходные данные будут выглядеть следующим образом:

```output
Name                                  Resource
------------------------------------  --------
5b198508-9712-43df-844b-977e5dfc30ea  SAPHANA
```

В ответе вы получите имя задания. Это имя можно использовать для отслеживания состояния задания с помощью командлета [az backup job show](/cli/azure/backup/job#az_backup_job_show).

## <a name="restore-as-files"></a>Восстановление в виде файлов

Чтобы восстановить данные резервной копии в виде файлов, а не базы данных, мы будем использовать режим восстановления **RestoreAsFiles**. Затем нужно выбрать предыдущую точку восстановления, например точку восстановления на определенный момент времени или любую другую. Когда файлы будут скопированы в указанное расположение, их можно будет использовать на любом компьютере SAP HANA, где их нужно восстановить в качестве базы данных. Так как эти файлы можно переместить на любой компьютер, теперь вы можете восстанавливать данные в разных подписках и регионах.

В этом руководстве показано, как использовать предыдущую точку восстановления на определенный момент времени (`28-11-2019-09:53:00`), а также расположение дампа файлов резервных копий как `/home/saphana/restoreasfiles` на том же сервере SAP HANA. Эту точку восстановления можно указать в следующих форматах: **дд-мм-гггг** и **дд-мм-гггг-чч:мм:сс**. Чтобы выбрать допустимый момент времени для восстановления, используйте командлет [az backup recoverypoint show-log-chain](/cli/azure/backup/recoverypoint#az_backup_recoverypoint_show_log_chain), который позволяет получить интервалы полных цепочек резервных копий журналов.

Используя указанные выше имя точки восстановления и режим восстановления, создайте объект конфигурации восстановления, выполнив командлет [az backup recoveryconfig show](/cli/azure/backup/recoveryconfig#az_backup_recoveryconfig_show). Давайте узнаем, что означают остальные параметры в этом командлете.

* **--target-container-name** обозначает имя сервера SAP HANA, который зарегистрирован в хранилище Служб восстановления и размещается в том же регионе, что и восстанавливаемая база данных. В этом руководстве показано, как восстановить базу данных в виде файлов на том же защищенном сервере SAP HANA с именем *hxehost*.
* **--rp-name.** Для восстановления до точки во времени точке восстановления будет присвоено имя **DefaultRangeRecoveryPoint**.

```azurecli-interactive
az backup recoveryconfig show --resource-group saphanaResourceGroup \
    --vault-name saphanaVault \
    --container-name VMAppContainer;Compute;saphanaResourceGroup;saphanaVM \
    --item-name saphanadatabase;hxe;hxe \
    --restore-mode RestoreAsFiles \
    --log-point-in-time 28-11-2019-09:53:00 \
    --rp-name DefaultRangeRecoveryPoint \
    --target-container-name VMAppContainer;Compute;saphanaResourceGroup;saphanaVM \
    --filepath /home/saphana/restoreasfiles \
    --output json
```

В ответ на приведенный выше запрос вы получите примерно такой объект конфигурации восстановления:

```output
{
  "alternate_directory_paths": null,
  "container_id": "/Subscriptions/ef4ab5a7-c2c0-4304-af80-af49f48af3d1/resourceGroups/saphanaResourceGroup/providers/Microsoft.RecoveryServices/vaults/SAPHANAVault/backupFabrics/Azure/protectionContainers/VMAppContainer;Compute;SAPHANA;hanamachine",
  "container_uri": "VMAppContainer;compute;saphana;hanamachine",
  "database_name": null,
  "filepath": "/home/",
  "item_type": "SAPHana",
  "item_uri": "SAPHanaDatabase;hxe;hxe",
  "log_point_in_time": "04-07-2020-09:53:00",
  "recovery_mode": "FileRecovery",
  "recovery_point_id": "DefaultRangeRecoveryPoint",
  "restore_mode": "AlternateLocation",
  "source_resource_id": "/subscriptions/ef4ab5a7-c2c0-4304-af80-af49f48af3d1/resourceGroups/saphanaResourceGroup/providers/Microsoft.Compute/virtualMachines/hanamachine"
}
```

Теперь выполните командлет [az restore restore-azurewl](/cli/azure/backup/restore#az_backup_restore_restore_azurewl) для восстановления базы данных в виде файлов. Чтобы использовать эту команду, мы предоставим приведенный выше код JSON, сохраненный в файл с именем *recoveryconfig.json*.

```azurecli-interactive
az backup restore restore-azurewl --resource-group saphanaResourceGroup \
    --vault-name saphanaVault \
    --restore-config recoveryconfig.json \
    --output json
```

Выходные данные будут выглядеть следующим образом:

```output
{
  "eTag": null,
  "id": "/Subscriptions/ef4ab5a7-c2c0-4304-af80-af49f48af3d1/resourceGroups/SAPHANARESOURCEGROUP/providers/Microsoft.RecoveryServices/vaults/SAPHANAVault/backupJobs/608e737e-c001-47ca-8c37-57d909c8a704",
  "location": null,
  "name": "608e737e-c001-47ca-8c37-57d909c8a704",
  "properties": {
    "actionsInfo": [
      "Cancellable"
    ],
    "activityId": "7ddd3c3a-c0eb-11ea-a5f8-54ee75ec272a",
    "backupManagementType": "AzureWorkload",
    "duration": "0:00:01.781847",
    "endTime": null,
    "entityFriendlyName": "HXE [hxehost]",
    "errorDetails": null,
    "extendedInfo": {
      "dynamicErrorMessage": null,
      "propertyBag": {
        "Job Type": "Restore as files"
      },
      "tasksList": [
        {
          "status": "InProgress",
          "taskId": "Transfer data from vault"
        }
      ]
    },
    "jobType": "AzureWorkloadJob",
    "operation": "Restore",
    "startTime": "2020-07-08T07:20:29.336434+00:00",
    "status": "InProgress",
    "workloadType": "SAPHanaDatabase"
  },
  "resourceGroup": "saphanaResourceGroup",
  "tags": null,
  "type": "Microsoft.RecoveryServices/vaults/backupJobs"
}
```

В ответе вы получите имя задания. Это имя можно использовать для отслеживания состояния задания с помощью командлета [az backup job show](/cli/azure/backup/job#az_backup_job_show).

Файлы, которые копируются в целевой контейнер:

* файлы резервной копии базы данных;
* файлы каталога;
* файлы метаданных JSON (для каждого задействованного файла резервной копии).

Как правило, использование пути к общей сетевой папке или пути к подключенной общей папке Azure, если он указан в качестве пути назначения, упрощает доступ к этим файлам с других компьютеров в той же сети или с подключенной к ним общей папкой Azure.

>[!NOTE]
>Чтобы восстановить файлы резервной копии базы данных в общей папке Azure, подключенной к целевой зарегистрированной виртуальной машине, убедитесь, что у привилегированной учетной записи есть разрешения на чтение и запись в общей папке Azure.

В зависимости от типа выбранной точки восстановления (**на определенный момент времени** или **полное и разностное восстановления**) вы увидите одну или несколько папок, созданных в целевом расположении. Одна из папок с именем `Data_<date and time of restore>` содержит полные и разностные резервные копии, а другая папка с именем `Log` — резервные копии журналов.

Переместите эти восстановленные файлы на сервер SAP HANA, где они будут восстановлены в качестве базы данных. Затем выполните следующие действия для восстановления базы данных:

1. Задайте разрешения для папки или каталога, в которых хранятся файлы резервных копий, с помощью следующей команды.

    ```bash
    chown -R <SID>adm:sapsys <directory>
    ```

1. Выполните следующий набор команд как `<SID>adm`.

    ```bash
    su - <sid>adm
    ```

1. Создайте файл каталога для восстановления. Извлеките значение **BackupId** из файла метаданных JSON для полной резервной копии. Оно будет использоваться далее в операции восстановления. Убедитесь, что полные резервные копии и резервные копии журналов находятся в разных папках, и удалите файлы каталогов и файлы метаданных JSON в этих папках.

    ```bash
    hdbbackupdiag --generate --dataDir <DataFileDir> --logDirs <LogFilesDir> -d <PathToPlaceCatalogFile>
    ```

    В приведенной выше команде:

    * `<DataFileDir>` — папка, содержащая полные резервные копии;
    * `<LogFilesDir>` — папка, содержащая резервные копии журналов;
    * `<PathToPlaceCatalogFile>` — папка, в которой должен быть размещен созданный файл каталога.

1. Выполните восстановление с использованием созданного файла каталога: в HANA Studio или с запросом на восстановление HDBSQL. Запросы HDBSQL перечислены ниже.

    * Восстановление на определенный момент времени:

        Если вы создаете новую восстановленную базу данных, выполните команду HDBSQL, чтобы создать новую базу данных `<DatabaseName>`, а затем прервите работу базы данных для восстановления. Но если восстанавливается только существующая база данных, выполните команду HDBSQL, чтобы прервать работу базы данных.

        Затем выполните следующую команду для восстановления базы данных:

        ```hdbsql
        RECOVER DATABASE FOR <DatabaseName> UNTIL TIMESTAMP '<TimeStamp>' CLEAR LOG USING SOURCE '<DatabaseName@HostName>'  USING CATALOG PATH ('<PathToGeneratedCatalogInStep3>') USING LOG PATH (' <LogFileDir>') USING DATA PATH ('<DataFileDir>') USING BACKUP_ID <BackupIdFromJsonFile> CHECK ACCESS USING FILE
        ```

        * `<DatabaseName>` — имя новой или существующей базы данных, которую необходимо восстановить.
        * `<Timestamp>` — точная метка времени для восстановления до точки во времени.
        * `<DatabaseName@HostName>` — имя базы данных, резервная копия которой используется для восстановления, и имя **узла** или сервера SAP HANA, на котором находится эта база данных. Параметр `USING SOURCE <DatabaseName@HostName>` определяет, что резервная копия данных (используемая для восстановления) относится к базе данных с другим идентификатором безопасности или именем, отличающимся от целевого компьютера SAP HANA. Поэтому для восстановления на том же сервере HANA, с которого выполняется резервное копирование, его не нужно указывать.
        * `<PathToGeneratedCatalogInStep3>` — путь к файлу каталога, созданному на **шаге 3**.
        * `<DataFileDir>` — папка, содержащая полные резервные копии.
        * `<LogFilesDir>` — папка, содержащая резервные копии журналов.
        * `<BackupIdFromJsonFile>` — значение **BackupId**, извлеченное на **шаге 3**.

    * Восстановление до определенной полной или разностной резервной копии:

        Если вы создаете новую восстановленную базу данных, выполните команду HDBSQL, чтобы создать новую базу данных `<DatabaseName>`, а затем прервите работу базы данных для восстановления. Но если восстанавливается только существующая база данных, выполните команду HDBSQL, чтобы прервать работу базы данных:

        ```hdbsql
        RECOVER DATA FOR <DatabaseName> USING BACKUP_ID <BackupIdFromJsonFile> USING SOURCE '<DatabaseName@HostName>'  USING CATALOG PATH ('<PathToGeneratedCatalogInStep3>') USING DATA PATH ('<DataFileDir>')  CLEAR LOG
        ```

        * `<DatabaseName>` — имя новой или существующей базы данных, которую необходимо восстановить.
        * `<Timestamp>` — точная метка времени для восстановления до точки во времени.
        * `<DatabaseName@HostName>` — имя базы данных, резервная копия которой используется для восстановления, и имя **узла** или сервера SAP HANA, на котором находится эта база данных. Параметр `USING SOURCE <DatabaseName@HostName>` определяет, что резервная копия данных (используемая для восстановления) относится к базе данных с другим идентификатором безопасности или именем, отличающимся от целевого компьютера SAP HANA. Поэтому для восстановления на том же сервере HANA, с которого выполняется резервное копирование, его не нужно указывать.
        * `<PathToGeneratedCatalogInStep3>` — путь к файлу каталога, созданному на **шаге 3**.
        * `<DataFileDir>` — папка, содержащая полные резервные копии.
        * `<LogFilesDir>` — папка, содержащая резервные копии журналов.
        * `<BackupIdFromJsonFile>` — значение **BackupId**, извлеченное на **шаге 3**.

## <a name="next-steps"></a>Дальнейшие действия

* Чтобы узнать, как управлять базами данных SAP HANA, для которых выполняется резервное копирование с помощью Azure CLI, перейдите к [этому руководству](tutorial-sap-hana-backup-cli.md).

* Сведения о восстановлении базы данных SAP HANA, работающей на виртуальной машине Azure, с помощью портал Azure см. в [этой статье](./sap-hana-db-restore.md).
