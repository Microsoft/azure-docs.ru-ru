---
title: Резервное копирование файловых ресурсов Azure с помощью Azure CLI
description: Узнайте, как использовать Azure CLI для резервного копирования файловых ресурсов Azure в хранилище служб восстановления.
ms.topic: conceptual
ms.date: 01/14/2020
ms.openlocfilehash: 34eea8daa6a0a8920c842178664055838b06a78a
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94565897"
---
# <a name="back-up-azure-file-shares-with-azure-cli"></a>Резервное копирование файловых ресурсов Azure с помощью Azure CLI

Интерфейс командной строки Azure (CLI) предоставляет возможности командной строки для управления ресурсами Azure. Это отличный инструмент для создания пользовательской автоматизации для использования ресурсов Azure. В этой статье подробно описано, как создавать резервные копии файловых ресурсов Azure с помощью Azure CLI. Эти действия также можно выполнить с помощью [Azure PowerShell](./backup-azure-afs-automation.md) или [портала Azure](backup-afs.md).

По завершении работы с этим руководством вы узнаете, как выполнять следующие операции с Azure CLI.

* Создание хранилища Служб восстановления
* Включение резервного копирования для файловых ресурсов Azure
* Активация резервной копии по запросу для файловых ресурсов

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

 - Для работы с этим руководством требуется Azure CLI версии 2.0.18 или более поздней. Если вы используете Azure Cloud Shell, последняя версия уже установлена.

## <a name="create-a-recovery-services-vault"></a>Создание хранилища Служб восстановления

Хранилище служб восстановления — это сущность, которая предоставляет объединенное представление и возможности управления для всех элементов резервного копирования. Когда выполняется задание резервного копирования для защищенного ресурса, в хранилище служб восстановления создается точка восстановления. Позже вы сможете использовать одну из этих точек восстановления, чтобы восстановить данные до определенной точки во времени.

Чтобы создать хранилище служб восстановления, выполните следующие действия.

1. Хранилище помещается в группу ресурсов. Если у вас нет группы ресурсов, создайте ее с помощью команды [AZ Group Create](/cli/azure/group#az-group-create) . В этом руководстве мы создадим новую группу ресурсов *azurefiles* в регионе "Восточная часть США".

    ```azurecli-interactive
    az group create --name AzureFiles --location eastus --output table
    ```

    ```output
    Location    Name
    ----------  ----------
    eastus      AzureFiles
    ```

1. Для создания хранилища используйте командлет [AZ Backup Vault Create](/cli/azure/backup/vault#az-backup-vault-create) . Разместите хранилище там же, где находится группа ресурсов.

    В следующем примере создается хранилище служб восстановления с именем *азурефилесваулт* в регионе Восточная часть США.

    ```azurecli-interactive
    az backup vault create --resource-group azurefiles --name azurefilesvault --location eastus --output table
    ```

    ```output
    Location    Name                ResourceGroup
    ----------  ----------------    ---------------
    eastus      azurefilesvault     azurefiles
    ```

## <a name="enable-backup-for-azure-file-shares"></a>Включение резервного копирования для файловых ресурсов Azure

В этом разделе предполагается, что у вас уже есть файловый ресурс Azure, для которого требуется настроить резервное копирование. Если у вас ее нет, создайте общую папку Azure с помощью команды [AZ Storage Share Create](/cli/azure/storage/share#az-storage-share-create) .

Чтобы включить резервное копирование для файловых ресурсов, необходимо создать политику защиты, определяющую время выполнения задания резервного копирования и время хранения точек восстановления. Политику архивации можно создать с помощью командлета [AZ Backup Policy Create](/cli/azure/backup/policy#az-backup-policy-create) .

В следующем примере используется командлет [AZ Backup Protection Enable-for-azurefileshare](/cli/azure/backup/protection#az-backup-protection-enable-for-azurefileshare) , чтобы включить резервное копирование для файлового ресурса *azurefiles* в учетной записи хранения *афсаккаунт* с помощью политики архивации по *расписанию 1* :

```azurecli-interactive
az backup protection enable-for-azurefileshare --vault-name azurefilesvault --resource-group  azurefiles --policy-name schedule1 --storage-account afsaccount --azure-file-share azurefiles  --output table
```

```output
Name                                  ResourceGroup
------------------------------------  ---------------
0caa93f4-460b-4328-ac1d-8293521dd928  azurefiles
```

Атрибут **Name** в выходных данных соответствует имени задания, созданного службой резервного копирования для операции **включения резервного копирования** . Чтобы отвести состояние задания, используйте командлет [AZ Backup Job](/cli/azure/backup/job#az-backup-job-show) #.

## <a name="trigger-an-on-demand-backup-for-file-share"></a>Активация резервной копии по запросу для файлового ресурса

Если вы хотите активировать резервное копирование по запросу для общей папки, а не ждать, пока политика архивации запустит задание в запланированное время, используйте командлет [AZ Backup Protection Backup-Now](/cli/azure/backup/protection#az-backup-protection-backup-now) .

Чтобы активировать резервное копирование по запросу, необходимо определить следующие параметры:

* **--Container-Name** — это имя учетной записи хранения, в которой размещена общая папка. Чтобы получить **имя** или **понятное имя** контейнера, используйте команду [AZ Backup Container List](/cli/azure/backup/container#az-backup-container-list) .
* **--Item-Name** — это имя общего файлового ресурса, для которого требуется активировать резервное копирование по запросу. Чтобы получить **имя** или **понятное имя** архивированного элемента, используйте команду [AZ Backup Item List](/cli/azure/backup/item#az-backup-item-list) .
* **--Сохранение** до указывает дату до тех пор, пока не будет храниться точка восстановления. Значение должно быть задано в формате времени в формате UTC (дд-мм-гггг).

Следующий пример активирует резервную копию по запросу для общей папки *azurefiles* в учетной записи хранения *афсаккаунт* с периодом хранения до *20-01-2020*.

```azurecli-interactive
az backup protection backup-now --vault-name azurefilesvault --resource-group azurefiles --container-name "StorageContainer;Storage;AzureFiles;afsaccount" --item-name "AzureFileShare;azurefiles" --retain-until 20-01-2020 --output table
```

```output
Name                                  ResourceGroup
------------------------------------  ---------------
9f026b4f-295b-4fb8-aae0-4f058124cb12  azurefiles
```

Атрибут **Name** в выходных данных соответствует имени задания, созданного службой архивации для операции резервного копирования по требованию. Чтобы отвести состояние задания, используйте командлет [AZ Backup Job показывать](/cli/azure/backup/job#az-backup-job-show) .

## <a name="next-steps"></a>Дальнейшие действия

* Узнайте, как [восстановить файловые ресурсы Azure с помощью интерфейса командной строки](restore-afs-cli.md) .
* Узнайте, как [управлять резервным копированием файловых ресурсов Azure с помощью интерфейса командной строки](manage-afs-backup-cli.md)
