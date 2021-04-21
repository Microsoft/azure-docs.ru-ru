---
title: Учебник. Восстановление виртуальной машины с помощью Azure CLI
description: Дополнительные сведения о восстановлении диска и создании восстановленной виртуальной машины в Azure с помощью служб архивации и восстановления.
ms.topic: tutorial
ms.date: 01/31/2019
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: 7f4d70f43f76c3a72cd8e53037d06d32e61c3cdb
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107768519"
---
# <a name="restore-a-vm-with-azure-cli"></a>Восстановление виртуальной машины с помощью Azure CLI

Служба архивации Azure создает точки восстановления, которые хранятся в геоизбыточных хранилищах восстановления. Используя точку восстановления, можно восстановить всю виртуальную машину или только отдельные файлы. В этой статье описан процесс восстановления всей виртуальной машины с помощью CLI. Из этого руководства вы узнаете, как выполнить следующие задачи:

> [!div class="checklist"]
>
> * Составление списка и выбор точек восстановления
> * Восстановление диска из точки восстановления
> * Создание виртуальной машины на основе восстановленного диска

Сведения об использовании PowerShell для восстановления диска и создания восстановленной виртуальной машины см. в разделе о [резервном копировании и восстановление виртуальных машин Azure с помощью PowerShell](backup-azure-vms-automation.md#restore-an-azure-vm).

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

 - Для работы с этим руководством требуется Azure CLI версии 2.0.18 или более поздней. Если вы используете Azure Cloud Shell, последняя версия уже установлена.

 - Для выполнения этого руководства требуется виртуальная машина Linux, защищенная с помощью службы архивации Azure. Для имитации случайного удаления виртуальной машины и процесса восстановления виртуальная машина создается на основе диска в точке восстановления. Если вам требуется виртуальная машина Linux, защищенная с помощью службы архивации Azure, см. раздел [Резервное копирование виртуальной машины в Azure с помощью интерфейса командной строки](quick-backup-vm-cli.md).

## <a name="backup-overview"></a>Обзор службы Azure Backup

Когда система Azure запускает архивацию, расширение резервного копирования на виртуальной машине делает моментальный снимок на момент времени. Расширение резервного копирования устанавливается на виртуальной машине при запросе первой резервной копии. Если на момент архивации виртуальная машина не запущена, Azure Backup также может создать моментальный снимок базового хранилища.

По умолчанию служба архивации Azure создает резервную копию, согласованную с файловой системой. После создания моментального снимка службой архивации Azure данные передаются в хранилище служб восстановления. Для повышения эффективности служба архивации Azure анализирует блоки данных и передает только те из них, которые были изменены с момента предыдущего резервного копирования.

После передачи данных служба удаляет моментальный снимок и создает точку восстановления.

## <a name="list-available-recovery-points"></a>Список доступных точек восстановления

Чтобы восстановить диск, выберите точку восстановления в качестве источника данных восстановления. Поскольку политика по умолчанию создает точку восстановления каждый день и хранит все точки в течение 30 дней, вы можете сохранить набор точек восстановления, а затем выбрать определенный момент времени для восстановления.

Чтобы просмотреть список доступных точек восстановления, используйте команду [az backup recoverypoint list](/cli/azure/backup/recoverypoint#az_backup_recoverypoint_list). **Имя** точки восстановления используется для восстановления дисков. В рамках этого руководства нам требуется последняя доступная точка восстановления. Параметр `--query [0].name` выбирает имя самой последней точки восстановления следующим образом:

```azurecli-interactive
az backup recoverypoint list \
    --resource-group myResourceGroup \
    --vault-name myRecoveryServicesVault \
    --backup-management-type AzureIaasVM \
    --container-name myVM \
    --item-name myVM \
    --query [0].name \
    --output tsv
```

## <a name="restore-a-vm-disk"></a>Восстановление диска виртуальной машины

> [!IMPORTANT]
> Мы настоятельно рекомендуем использовать Azure CLI версии 2.0.74 или более поздней, чтобы получить все преимущества быстрого восстановления, в том числе восстановление управляемых дисков. Лучше всегда использовать последнюю версию.

### <a name="managed-disk-restore"></a>Восстановление управляемого диска

Если в виртуальной машине, для которой используется резервное копирование, есть управляемые диски и вы намерены восстанавливать эти управляемые диски из точки восстановления, вам необходимо предоставить учетную запись хранения Azure. Эта учетная запись хранения используется для сохранения конфигурации виртуальной машины и шаблона развертывания, которые позже можно применить для развертывания виртуальной машины с восстановленных дисков. Также предоставьте целевую группу ресурсов для восстановления управляемых дисков.

1. Создайте учетную запись хранения с помощью команды [az storage account create](/cli/azure/storage/account#az_storage_account_create). Имя учетной записи хранения следует указывать в нижнем регистре, оно должно быть глобально уникальным. Замените *mystorageaccount* собственным уникальным именем:

    ```azurecli-interactive
    az storage account create \
        --resource-group myResourceGroup \
        --name mystorageaccount \
        --sku Standard_LRS
    ```

2. Восстановите диск из точки восстановления с помощью команды [az backup restore restore-disks](/cli/azure/backup/restore#az_backup_restore_restore_disks). Замените *mystorageaccount* именем учетной записи хранения, созданной с помощью предыдущей команды. Замените *myRecoveryPointName* именем точки восстановления, полученной в выходных данных предыдущей команды [az backup recoverypoint list](/cli/azure/backup/recoverypoint#az_backup_recoverypoint_list). ***Также предоставьте целевую группу ресурсов, в которую будут восстановлены управляемые диски***.

    ```azurecli-interactive
    az backup restore restore-disks \
        --resource-group myResourceGroup \
        --vault-name myRecoveryServicesVault \
        --container-name myVM \
        --item-name myVM \
        --storage-account mystorageaccount \
        --rp-name myRecoveryPointName \
        --target-resource-group targetRG
    ```

    > [!WARNING]
    > Если параметр **target-resource-group** не указан, управляемые диски будут восстановлены в предоставленную учетную запись в виде неуправляемых дисков. Это существенно повлияет на время восстановления, поскольку время на восстановление дисков полностью зависит от используемой учетной записи хранения. Вы можете использовать мгновенное восстановление, только если указан параметр target-resource-group. Если вы хотите восстановить управляемые диски как неуправляемые, не указывайте параметр **target-resource-group**. Вместо этого укажите параметр **restore-as-unmanaged-disk**, как показано ниже. Этот параметр доступен, начиная с версии 3.4.0 az.

    ```azurecli-interactive
    az backup restore restore-disks \
    --resource-group myResourceGroup \
    --vault-name myRecoveryServicesVault \
    --container-name myVM \
    --item-name myVM \
    --storage-account mystorageaccount \
    --rp-name myRecoveryPointName \
    --restore-as-unmanaged-disk
    ```

Это восстановит для данной учетной записи хранения управляемые диски как неуправляемые, и функция "мгновенного" восстановления использоваться не будет. В будущих версиях CLI необходимо будет обязательно указывать параметр **target-resource-group** или параметр **restore-as-unmanaged-disk**.

### <a name="unmanaged-disks-restore"></a>Восстановление неуправляемых дисков

Если в виртуальной машине, для которой используется резервное копирование, есть неуправляемые диски и вы намерены восстанавливать их из точки восстановления, вам необходимо предоставить учетную запись хранения Azure. Эта учетная запись хранения используется для сохранения конфигурации виртуальной машины и шаблона развертывания, которые позже можно применить для развертывания виртуальной машины с восстановленных дисков. По умолчанию неуправляемые диски восстанавливаются в исходных учетных записях хранения. Если нужно восстановить все неуправляемые диски в одном месте, указанная учетная запись хранения может также использоваться в качестве промежуточного расположения для этих дисков.

Затем этот восстановленный диск используется для создания виртуальной машины.

1. Создайте учетную запись хранения с помощью команды [az storage account create](/cli/azure/storage/account#az_storage_account_create). Имя учетной записи хранения следует указывать в нижнем регистре, оно должно быть глобально уникальным. Замените *mystorageaccount* собственным уникальным именем:

    ```azurecli-interactive
    az storage account create \
        --resource-group myResourceGroup \
        --name mystorageaccount \
        --sku Standard_LRS
    ```

2. Восстановите диск из точки восстановления с помощью команды [az backup restore restore-disks](/cli/azure/backup/restore#az_backup_restore_restore_disks). Замените *mystorageaccount* именем учетной записи хранения, созданной с помощью предыдущей команды. Замените *myRecoveryPointName* именем точки восстановления, полученной в выходных данных предыдущей команды [az backup recoverypoint list](/cli/azure/backup/recoverypoint#az_backup_recoverypoint_list):

    ```azurecli-interactive
    az backup restore restore-disks \
        --resource-group myResourceGroup \
        --vault-name myRecoveryServicesVault \
        --container-name myVM \
        --item-name myVM \
        --storage-account mystorageaccount \
        --rp-name myRecoveryPointName
    ```

Как уже упоминалось, неуправляемые диски восстанавливаются в исходных учетных записях хранения. Это обеспечивает наилучшую производительность восстановления. Но если вам нужно восстановить все неуправляемые диски в указанную учетную запись хранения, используйте соответствующий флаг, как показано ниже.

```azurecli-interactive
    az backup restore restore-disks \
        --resource-group myResourceGroup \
        --vault-name myRecoveryServicesVault \
        --container-name myVM \
        --item-name myVM \
        --storage-account mystorageaccount \
        --rp-name myRecoveryPointName \
        --restore-to-staging-storage-account
    ```

## Monitor the restore job

To monitor the status of restore job, use [az backup job list](/cli/azure/backup/job#az_backup_job_list):

```azurecli-interactive
az backup job list \
    --resource-group myResourceGroup \
    --vault-name myRecoveryServicesVault \
    --output table
```

В результате вы получите приблизительно следующие выходные данные. Они указывают на то, что задание восстановления *выполняется*:

```output
Name      Operation        Status      Item Name    Start Time UTC       Duration
--------  ---------------  ----------  -----------  -------------------  --------------
7f2ad916  Restore          InProgress  myvm         2017-09-19T19:39:52  0:00:34.520850
a0a8e5e6  Backup           Completed   myvm         2017-09-19T03:09:21  0:15:26.155212
fe5d0414  ConfigureBackup  Completed   myvm         2017-09-19T03:03:57  0:00:31.191807
```

Параметр *Состояние* для задания восстановления принимает значение *Завершено*, это когда вся необходимая информация (конфигурация виртуальной машины и шаблон развертывания) восстановлена в учетной записи хранения.

## <a name="create-a-vm-from-the-restored-disk"></a>Создание виртуальной машины на основе восстановленного диска

Последним шагом является создание виртуальной машины на основе восстановленных дисков. Для создания виртуальной машины можно использовать шаблон развертывания, скачанный в указанную учетную запись хранения.

### <a name="fetch-the-job-details"></a>Получение подробных сведений о задании

В подробных сведениях о результирующем задании указан URI шаблона, который можно получить и развернуть. Используйте команду "job show", чтобы получить дополнительные сведения о запущенном задании восстановления.

```azurecli-interactive
az backup job show \
    -v myRecoveryServicesVault \
    -g myResourceGroup \
    -n 1fc2d55d-f0dc-4ca6-ad48-aca0fe5d0414
```

Выходные данные этого запроса содержат подробные сведения, из которых нас интересует только содержимое учетной записи хранения. Для получения нужных сведений мы можем использовать [функцию запросов](/cli/azure/query-azure-cli) в Azure CLI.

```azurecli-interactive
az backup job show \
    -v myRecoveryServicesVault \
    -g myResourceGroup \
    -n 1fc2d55d-f0dc-4ca6-ad48-aca0fe5d0414 \
    --query properties.extendedInfo.propertyBag

{
  "Config Blob Container Name": "myVM-daa1931199fd4a22ae601f46d8812276",
  "Config Blob Name": "config-myVM-1fc2d55d-f0dc-4ca6-ad48-aca0fe5d0414.json",
  "Config Blob Uri": "https://mystorageaccount.blob.core.windows.net/myVM-daa1931199fd4a22ae601f46d8812276/config-appvm8-1fc2d55d-f0dc-4ca6-ad48-aca0519c0232.json",
  "Job Type": "Recover disks",
  "Recovery point time ": "12/25/2019 10:07:11 PM",
  "Target Storage Account Name": "mystorageaccount",
  "Target resource group": "mystorageaccountRG",
  "Template Blob Uri": "https://mystorageaccount.blob.core.windows.net/myVM-daa1931199fd4a22ae601f46d8812276/azuredeploy1fc2d55d-f0dc-4ca6-ad48-aca0519c0232.json"
}
```

### <a name="fetch-the-deployment-template"></a>Получение шаблона развертывания

Этот шаблон недоступен напрямую, так как он находится в учетной записи хранения клиента, в указанном контейнере. Для доступа к этому шаблону требуется полный URL-адрес (вместе с временным маркером SAS).

Сначала извлеките URI BLOB-объекта шаблона из сведений о задании.

```azurecli-interactive
az backup job show \
    -v myRecoveryServicesVault \
    -g myResourceGroup \
    -n 1fc2d55d-f0dc-4ca6-ad48-aca0fe5d0414 \
    --query properties.extendedInfo.propertyBag."""Template Blob Uri"""

"https://mystorageaccount.blob.core.windows.net/myVM-daa1931199fd4a22ae601f46d8812276/azuredeploy1fc2d55d-f0dc-4ca6-ad48-aca0519c0232.json"
```

URI BLOB-объекта шаблона имеет указанный ниже формат, из которого вы можете извлечь имя шаблона.

```https
https://<storageAccountName.blob.core.windows.net>/<containerName>/<templateName>
```

Например, в приведенном выше примере имя шаблона имеет значение ```azuredeploy1fc2d55d-f0dc-4ca6-ad48-aca0519c0232.json```, а имя контейнера — ```myVM-daa1931199fd4a22ae601f46d8812276```.

Теперь получите маркер SAS для этого контейнера и шаблона, как описано [здесь](../azure-resource-manager/templates/secure-template-with-sas-token.md?tabs=azure-cli#provide-sas-token-during-deployment).

```azurecli-interactive
expiretime=$(date -u -d '30 minutes' +%Y-%m-%dT%H:%MZ)
connection=$(az storage account show-connection-string \
    --resource-group mystorageaccountRG \
    --name mystorageaccount \
    --query connectionString)
token=$(az storage blob generate-sas \
    --container-name myVM-daa1931199fd4a22ae601f46d8812276 \
    --name azuredeploy1fc2d55d-f0dc-4ca6-ad48-aca0519c0232.json \
    --expiry $expiretime \
    --permissions r \
    --output tsv \
    --connection-string $connection)
url=$(az storage blob url \
   --container-name myVM-daa1931199fd4a22ae601f46d8812276 \
    --name azuredeploy1fc2d55d-f0dc-4ca6-ad48-aca0519c0232.json \
    --output tsv \
    --connection-string $connection)
```

### <a name="deploy-the-template-to-create-the-vm"></a>Развертывание шаблона для создания виртуальной машины

Теперь разверните шаблон, чтобы создать виртуальную машину, как описано [здесь](../azure-resource-manager/templates/deploy-cli.md).

```azurecli-interactive
az deployment group create \
  --resource-group ExampleGroup \
  --template-uri $url?$token
```

Чтобы убедиться, что виртуальная машина была создана на основе восстановленного диска, перечислите виртуальные машины в группе ресурсов с помощью команды [az vm list](/cli/azure/vm#az_vm_list) следующим образом:

```azurecli-interactive
az vm list --resource-group myResourceGroup --output table
```

## <a name="next-steps"></a>Дальнейшие действия

В этом руководстве описывается восстановление диска из точки восстановления, а затем создание виртуальной машины на его основе. Вы ознакомились с выполнением следующих задач:

> [!div class="checklist"]
>
> * Составление списка и выбор точек восстановления
> * Восстановление диска из точки восстановления
> * Создание виртуальной машины на основе восстановленного диска

Перейдите к следующему руководству, чтобы узнать о восстановлении отдельных файлов из точки восстановления.

> [!div class="nextstepaction"]
> [Восстановление файлов на виртуальной машине в Azure](tutorial-restore-files.md)
