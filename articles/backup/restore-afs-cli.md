---
title: Восстановление файловых ресурсов Azure с помощью Azure CLI
description: Узнайте, как с помощью Azure CLI восстановить резервные копии файловых ресурсов Azure в хранилище служб восстановления.
ms.topic: conceptual
ms.date: 01/16/2020
ms.openlocfilehash: a025de7bfb9db037b2008d69be7782feabb482f3
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94562327"
---
# <a name="restore-azure-file-shares-with-the-azure-cli"></a>Восстановление файловых ресурсов Azure с помощью Azure CLI

Azure CLI предоставляет возможности командной строки для управления ресурсами Azure. Это отличный инструмент для создания пользовательской автоматизации для использования ресурсов Azure. В этой статье объясняется, как восстановить всю общую папку или отдельные файлы из точки восстановления, созданной [Azure Backup](./backup-overview.md) с помощью Azure CLI. Эти действия также можно выполнить с помощью [Azure PowerShell](./backup-azure-afs-automation.md) или [портала Azure](backup-afs.md).

В конце этой статьи вы узнаете, как выполнять следующие операции с Azure CLI.

* Просмотр точек восстановления для резервной копии файлового ресурса Azure.
* Восстановите полную общую папку Azure.
* Восстановление отдельных файлов или папок.

>[!NOTE]
> Azure Backup теперь поддерживает восстановление нескольких файлов или папок в исходном или альтернативном расположении с помощью Azure CLI. Дополнительные сведения см. в разделе [Восстановление нескольких файлов или папок в исходное или альтернативное расположение](#restore-multiple-files-or-folders-to-original-or-alternate-location) этого документа.

## <a name="prerequisites"></a>Предварительные условия

В этой статье предполагается, что у вас уже есть файловый ресурс Azure, для которого создана резервная копия Azure Backup. Если у вас ее нет, см. раздел [резервное копирование файловых ресурсов Azure с помощью интерфейса командной строки](backup-afs-cli.md) для настройки резервного копирования для общей папки. В этой статье вы используете следующие ресурсы:

| Общая папка | Учетная запись хранения | Регион | Сведения |
|---|---|---|---|
| *azurefiles* | *афсаккаунт* | EastUS | Резервное копирование исходного исходного кода с помощью Azure Backup |
| *azurefiles1* | *afaccount1* | EastUS | Источник назначения, используемый для восстановления альтернативного расположения |

Вы можете использовать ту же структуру для файловых ресурсов, чтобы испытать различные типы восстановлений, описанные в этой статье.

[!INCLUDE [azure-cli-prepare-your-environment-h3.md](../../includes/azure-cli-prepare-your-environment-h3.md)]

 - Для работы с этим руководством требуется Azure CLI версии 2.0.18 или более поздней. Если вы используете Azure Cloud Shell, последняя версия уже установлена.

## <a name="fetch-recovery-points-for-the-azure-file-share"></a>Получение точек восстановления для файлового ресурса Azure

Используйте командлет [AZ Backup recoverypoint List](/cli/azure/backup/recoverypoint#az-backup-recoverypoint-list) , чтобы получить список всех точек восстановления для архивированного файлового ресурса.

В следующем примере извлекается список точек восстановления для общей папки *azurefiles* в учетной записи хранения *афсаккаунт* .

```azurecli-interactive
az backup recoverypoint list --vault-name azurefilesvault --resource-group azurefiles --container-name "StorageContainer;Storage;AzureFiles;afsaccount" --backup-management-type azurestorage --item-name "AzureFileShare;azurefiles" --workload-type azurefileshare --out table
```

Вы также можете запустить предыдущий командлет, используя понятное имя для контейнера и элемента, предоставив следующие два дополнительных параметра:

* **--BACKUP-Management-Type**: *azurestorage*
* **--Рабочая нагрузка — тип**: *azurefileshare*

```azurecli-interactive
az backup recoverypoint list --vault-name azurefilesvault --resource-group azurefiles --container-name afsaccount --backup-management-type azurestorage --item-name azurefiles --workload-type azurefileshare --out table
```

Результирующий набор — это список точек восстановления со сведениями о времени и согласованности для каждой точки восстановления.

```output
Name                Time                        Consistency
------------------  -------------------------   --------------------
932887541532871865  2020-01-05T07:08:23+00:00   FileSystemConsistent
932885927361238054  2020-01-05T07:08:10+00:00   FileSystemConsistent
932879614553967772  2020-01-04T21:33:04+00:00   FileSystemConsistent
```

Атрибут **Name** в выходных данных соответствует имени точки восстановления, которое можно использовать в качестве значения параметра **--RP-Name** в операциях восстановления.

## <a name="full-share-recovery-by-using-the-azure-cli"></a>Полное восстановление общего ресурса с помощью Azure CLI

С помощью этого параметра можно восстановить полную общую папку в исходном или альтернативном расположении.

Определите следующие параметры для выполнения операций восстановления:

* **--Container-Name**: имя учетной записи хранения, в которой размещена резервная копия исходной общей папки. Чтобы получить имя или понятное имя контейнера, используйте команду [AZ Backup Container List](/cli/azure/backup/container#az-backup-container-list) .
* **--Item-Name**: имя резервной копии исходной общей папки, которую вы хотите использовать для операции восстановления. Чтобы получить имя или понятное имя архивированного элемента, используйте команду [AZ Backup Item List](/cli/azure/backup/item#az-backup-item-list) .

### <a name="restore-a-full-share-to-the-original-location"></a>Восстановление полной общей папки в исходное расположение

При восстановлении в исходное расположение не нужно указывать параметры, связанные с целевым объектом. Необходимо предоставить только **разрешение конфликтов** .

В следующем примере используется командлет [AZ Backup Restore-azurefileshare](/cli/azure/backup/restore#az-backup-restore-restore-azurefileshare) с параметром Restore Mode, имеющим значение *оригиналлокатион* для восстановления общей папки *azurefiles* в исходном расположении. Вы используете точку восстановления 932883129628959823, полученную при [получении точек восстановления для файлового ресурса Azure](#fetch-recovery-points-for-the-azure-file-share):

```azurecli-interactive
az backup restore restore-azurefileshare --vault-name azurefilesvault --resource-group azurefiles --rp-name 932887541532871865   --container-name "StorageContainer;Storage;AzureFiles;afsaccount" --item-name "AzureFileShare;azurefiles" --restore-mode originallocation --resolve-conflict overwrite --out table
```

```output
Name                                  ResourceGroup
------------------------------------  ---------------
6a27cc23-9283-4310-9c27-dcfb81b7b4bb  azurefiles
```

Атрибут **Name** в выходных данных соответствует имени задания, созданного службой резервного копирования для операции восстановления. Для наблюдения за состоянием задания используйте команду [AZ Backup Job показывать](/cli/azure/backup/job#az-backup-job-show) командлет.

### <a name="restore-a-full-share-to-an-alternate-location"></a>Восстановление полного общего ресурса в альтернативное расположение

Этот параметр можно использовать для восстановления общей папки в альтернативное расположение и сохранения исходного общего файлового ресурса как есть. Укажите следующие параметры для восстановления в альтернативном расположении:

* **--Target-Storage-Account**— учетная запись хранения, в которую восстанавливается резервное содержимое. Целевая учетная запись хранения должна быть в том же расположении, что и хранилище.
* **--Target-File-Share**: общая папка в целевой учетной записи хранения, в которую восстанавливается резервное содержимое.
* **--Target-Folder**— папка в общей папке, в которую восстанавливаются данные. Если содержимое резервной копии необходимо восстановить в корневую папку, укажите значения целевой папки в виде пустой строки.
* **--Resolve-конфликт**: инструкция при конфликте с восстановленными данными. Принимает **перезаписать** или **пропустить**.

В следующем примере используется [AZ Backup Restore-azurefileshare](/cli/azure/backup/restore#az-backup-restore-restore-azurefileshare) с режимом восстановления *алтернателокатион* для восстановления общей папки *azurefiles* в учетной записи хранения *афсаккаунт* в общую папку *azurefiles1* в учетной записи хранения *afaccount1* .

```azurecli-interactive
az backup restore restore-azurefileshare --vault-name azurefilesvault --resource-group azurefiles --rp-name 932883129628959823 --container-name "StorageContainer;Storage;AzureFiles;afsaccount" --item-name "AzureFileShare;azurefiles" --restore-mode alternatelocation --target-storage-account afaccount1 --target-file-share azurefiles1 --target-folder restoredata --resolve-conflict overwrite --out table
```

```output
Name                                  ResourceGroup
------------------------------------  ---------------
babeb61c-d73d-4b91-9830-b8bfa83c349a  azurefiles
```

Атрибут **Name** в выходных данных соответствует имени задания, созданного службой резервного копирования для операции восстановления. Для наблюдения за состоянием задания используйте команду [AZ Backup Job показывать](/cli/azure/backup/job#az-backup-job-show) командлет.

## <a name="item-level-recovery"></a>Восстановление на уровне элементов

С помощью этого параметра можно восстановить отдельные файлы или папки в исходном или альтернативном расположении.

Определите следующие параметры для выполнения операций восстановления:

* **--Container-Name**: имя учетной записи хранения, в которой размещена резервная копия исходной общей папки. Чтобы получить имя или понятное имя контейнера, используйте команду [AZ Backup Container List](/cli/azure/backup/container#az-backup-container-list) .
* **--Item-Name**: имя резервной копии исходной общей папки, которую вы хотите использовать для операции восстановления. Чтобы получить имя или понятное имя архивированного элемента, используйте команду [AZ Backup Item List](/cli/azure/backup/item#az-backup-item-list) .

Укажите следующие параметры для элементов, которые требуется восстановить:

* **SourceFilePath**: абсолютный путь к файлу, восстанавливаемому в общей папке, в виде строки. Это тот же путь, который используется в команде [AZ Storage File Download](/cli/azure/storage/file#az-storage-file-download) или [AZ Storage File показывать](/cli/azure/storage/file#az-storage-file-show) команды CLI.
* **Саурцефилетипе**: выберите, выбран ли каталог или файл. Принимает **Каталог** или **файл**.
* **ResolveConflict**: инструкция при конфликте с восстановленными данными. Принимает **перезаписать** или **пропустить**.

### <a name="restore-individual-files-or-folders-to-the-original-location"></a>Восстановление отдельных файлов или папок в исходное расположение

Используйте командлет [AZ Backup Restore-azurefiles](/cli/azure/backup/restore#az-backup-restore-restore-azurefiles) с параметром Restore Mode, имеющим значение *оригиналлокатион* , чтобы восстановить определенные файлы или папки в исходное расположение.

В следующем примере файл *RestoreTest.txt* восстанавливается в исходном расположении: файловый ресурс *azurefiles* .

```azurecli-interactive
az backup restore restore-azurefiles --vault-name azurefilesvault --resource-group azurefiles --rp-name 932881556234035474 --container-name "StorageContainer;Storage;AzureFiles;afsaccount" --item-name "AzureFileShare;azurefiles" --restore-mode originallocation  --source-file-type file --source-file-path "Restore/RestoreTest.txt" --resolve-conflict overwrite  --out table
```

```output
Name                                  ResourceGroup
------------------------------------  ---------------
df4d9024-0dcb-4edc-bf8c-0a3d18a25319  azurefiles
```

Атрибут **Name** в выходных данных соответствует имени задания, созданного службой резервного копирования для операции восстановления. Для наблюдения за состоянием задания используйте команду [AZ Backup Job показывать](/cli/azure/backup/job#az-backup-job-show) командлет.

### <a name="restore-individual-files-or-folders-to-an-alternate-location"></a>Восстановление отдельных файлов или папок в альтернативное расположение

Чтобы восстановить определенные файлы или папки в альтернативное расположение, используйте командлет [AZ Backup Restore-azurefiles](/cli/azure/backup/restore#az-backup-restore-restore-azurefiles) с параметром Restore Mode, имеющим значение *алтернателокатион* , и укажите следующие параметры, связанные с целевым объектом:

* **--Target-Storage-Account**— учетная запись хранения, в которую восстанавливается резервное содержимое. Целевая учетная запись хранения должна быть в том же расположении, что и хранилище.
* **--Target-File-Share**: общая папка в целевой учетной записи хранения, в которую восстанавливается резервное содержимое.
* **--Target-Folder**— папка в общей папке, в которую восстанавливаются данные. Если резервное содержимое должно быть восстановлено в корневую папку, присвойте значение целевой папки в виде пустой строки.

В следующем примере выполняется восстановление файла *RestoreTest.txt* , первоначально присутствующего в общей папке *azurefiles* , в альтернативное расположение: папка *ресторедата* в общей папке *azurefiles1* , размещенной в учетной записи хранения *afaccount1* .

```azurecli-interactive
az backup restore restore-azurefiles --vault-name azurefilesvault --resource-group azurefiles --rp-name 932881556234035474 --container-name "StorageContainer;Storage;AzureFiles;afsaccount" --item-name "AzureFileShare;azurefiles" --restore-mode alternatelocation --target-storage-account afaccount1 --target-file-share azurefiles1 --target-folder restoredata --resolve-conflict overwrite --source-file-type file --source-file-path "Restore/RestoreTest.txt" --out table
```

```output
Name                                  ResourceGroup
------------------------------------  ---------------
df4d9024-0dcb-4edc-bf8c-0a3d18a25319  azurefiles
```

Атрибут **Name** в выходных данных соответствует имени задания, созданного службой резервного копирования для операции восстановления. Для наблюдения за состоянием задания используйте команду [AZ Backup Job показывать](/cli/azure/backup/job#az-backup-job-show) командлет.

## <a name="restore-multiple-files-or-folders-to-original-or-alternate-location"></a>Восстановление нескольких файлов или папок в исходном или альтернативном расположении

Чтобы выполнить восстановление нескольких элементов, передайте значение параметра **Source-File-Path** в виде **разделенных пробелами** путей ко всем файлам или папкам, которые требуется восстановить.

В следующем примере выполняется восстановление *Restore.txt* и *AFS Report.docx* файлов в их исходном расположении.

```azurecli-interactive
az backup restore restore-azurefiles --vault-name azurefilesvault --resource-group azurefiles --rp-name 932889937058317910 --container-name "StorageContainer;Storage;AzureFiles;afsaccount" --item-name "AzureFileShare;azurefiles" --restore-mode originallocation  --source-file-type file --source-file-path "Restore Test.txt" "AFS Testing Report.docx" --resolve-conflict overwrite  --out table
```

Результат будет выглядеть примерно так:

```output
Name                                          ResourceGroup
------------------------------------          ---------------
649b0c14-4a94-4945-995a-19e2aace0305          azurefiles
```

Атрибут **Name** в выходных данных соответствует имени задания, созданного службой резервного копирования для операции восстановления. Для наблюдения за состоянием задания используйте команду [AZ Backup Job показывать](/cli/azure/backup/job#az-backup-job-show) командлет.

Если требуется восстановить несколько элементов в альтернативное расположение, используйте приведенную выше команду, указав параметры, связанные с целевым объектом, как описано в разделе [восстановление отдельных файлов или папок в альтернативное расположение](#restore-individual-files-or-folders-to-an-alternate-location) .

## <a name="next-steps"></a>Дальнейшие действия

Узнайте, как [управлять резервными копиями файловых ресурсов Azure с помощью Azure CLI](manage-afs-backup-cli.md).
