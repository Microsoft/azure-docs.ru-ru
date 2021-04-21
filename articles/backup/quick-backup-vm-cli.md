---
title: Краткое руководство. Резервное копирование виртуальной машины с помощью Azure CLI
description: В этом кратком руководстве вы узнаете, как создать хранилище Служб восстановления, включить защиту на виртуальной машине и создать начальную точку восстановления с помощью Azure CLI.
ms.devlang: azurecli
ms.topic: quickstart
ms.date: 01/31/2019
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: 521f9332590eca5cdf94fadc4b59ab4d482eb658
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107768565"
---
# <a name="back-up-a-virtual-machine-in-azure-with-the-azure-cli"></a>Резервное копирование виртуальной машины в Azure с помощью Azure CLI

Azure CLI используется для создания ресурсов Azure и управления ими из командной строки или с помощью скриптов. Для защиты данных можно создавать архивы с регулярным интервалом. Служба Azure Backup создает точки восстановления, которые могут храниться в геоизбыточных хранилищах служб восстановления. В этой статье объясняется, как создать резервную копию виртуальной машины в Azure с помощью Azure CLI. Эти действия также можно выполнить с помощью [Azure PowerShell](quick-backup-vm-powershell.md) или [портала Azure](quick-backup-vm-portal.md).

В этом руководстве объясняется, как включить резервное копирование существующей виртуальной машины Azure. Если вам необходимо создать виртуальную машину, см. статью [о создании виртуальной машины с помощью Azure CLI](../virtual-machines/linux/quick-create-cli.md).

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

 - Для работы с этим кратким руководством требуется Azure CLI версии 2.0.18 или более поздней. Если вы используете Azure Cloud Shell, последняя версия уже установлена.

## <a name="create-a-recovery-services-vault"></a>Создание хранилища Служб восстановления

Хранилище служб восстановления — это логический контейнер, в котором хранятся данные резервного копирования для каждого защищенного ресурса, например виртуальных машин Azure. Когда выполняется задание резервного копирования для защищенного ресурса, в хранилище служб восстановления создается точка восстановления. Позже вы сможете использовать одну из этих точек восстановления, чтобы восстановить данные до определенной точки во времени.

Создайте хранилище служб восстановления с помощью команды [az backup vault create](/cli/azure/backup/vault#az_backup_vault_create). Укажите те же группу ресурсов и расположение, что и для виртуальной машины, которую необходимо защитить. Если вы создали виртуальную машину по [этому краткому руководству](../virtual-machines/linux/quick-create-cli.md), вы имеете следующее:

- группу ресурсов с именем *myResourceGroup*;
- виртуальную машину с именем *myVM*;
- ресурсы в расположении *eastus*.

```azurecli-interactive
az backup vault create --resource-group myResourceGroup \
    --name myRecoveryServicesVault \
    --location eastus
```

По умолчанию в качестве хранилища служб восстановления задано геоизбыточное хранилище. Геоизбыточное хранилище гарантирует, что данные резервного копирования реплицируются во вторичный регион, который находится в сотнях километров от первичного региона. Если вам необходимо изменить параметр избыточности хранилища, используйте командлет [az backup vault backup-properties set](/cli/azure/backup/vault/backup-properties#az_backup_vault_backup_properties_set).

```azurecli
az backup vault backup-properties set \
    --name myRecoveryServicesVault  \
    --resource-group myResourceGroup \
    --backup-storage-redundancy "LocallyRedundant/GeoRedundant"
```

## <a name="enable-backup-for-an-azure-vm"></a>Включение резервного копирования для виртуальной машины Azure

Создайте политику защиты, чтобы определить, когда выполнять задание резервного копирования и как долго хранить точку восстановления. Согласно политике защиты по умолчанию задание резервного копирования выполняется каждый день, а точки восстановления хранятся в течение 30 дней. Вы можете использовать значения политики по умолчанию, чтобы быстро защитить виртуальную машину. Чтобы включить резервное копирование для виртуальной машины, выполните команду [az backup protection enable-for-vm](/cli/azure/backup/protection#az_backup_protection_enable_for_vm). Укажите группу ресурсов и виртуальную машину, которые нужно защитить, и выберите необходимую политику:

```azurecli-interactive
az backup protection enable-for-vm \
    --resource-group myResourceGroup \
    --vault-name myRecoveryServicesVault \
    --vm myVM \
    --policy-name DefaultPolicy
```

> [!NOTE]
> Если виртуальная машина размещена не в той же группе ресурсов, что и хранилище, myResourceGroup будет обращаться к той группе ресурсов, в которой создано хранилище. Вместо имени виртуальной машины введите ее идентификатор, как указано ниже.

```azurecli-interactive
az backup protection enable-for-vm \
    --resource-group myResourceGroup \
    --vault-name myRecoveryServicesVault \
    --vm $(az vm show -g VMResourceGroup -n MyVm --query id | tr -d '"') \
    --policy-name DefaultPolicy
```

> [!IMPORTANT]
> При использовании CLI для одновременного резервного копирования нескольких виртуальных машин убедитесь, что с одной политикой связано не более 100 виртуальных машин. Это [рекомендуемый метод](./backup-azure-vm-backup-faq.yml#is-there-a-limit-on-number-of-vms-that-can-be-associated-with-the-same-backup-policy). В настоящее время клиент PowerShell не выполняет явную блокировку при наличии более 100 виртуальных машин, но в будущем планируется добавить проверку.

## <a name="start-a-backup-job"></a>Запуск задания резервного копирования

Чтобы начать резервное копирование сейчас, а не ждать задания, которое запустится в запланированное время согласно политике по умолчанию, используйте команду [az backup protection backup-now](/cli/azure/backup/protection#az_backup_protection_backup_now). В ходе первого задания резервного копирования создается точка полного восстановления. Во всех заданиях после начального резервного копирования создаются добавочные точки восстановления. Добавочные точки восстановления требуют мало места и времени, так как они позволяют передать только изменения, внесенные с момента последнего резервного копирования.

Для резервного копирования виртуальной машины используются следующие параметры:

- `--container-name` — имя виртуальной машины;
- `--item-name` — имя виртуальной машины;
- `--retain-until` — задайте в качестве значения последнюю доступную дату в формате времени UTC (**дд-мм-гггг**), до которой должна храниться точка восстановления.

В следующем примере создается резервная копия виртуальной машины *myVM* и в качестве даты окончания срока действия для точки восстановления задается 18 октября 2017 г.

```azurecli-interactive
az backup protection backup-now \
    --resource-group myResourceGroup \
    --vault-name myRecoveryServicesVault \
    --container-name myVM \
    --item-name myVM \
    --retain-until 18-10-2017
```

## <a name="monitor-the-backup-job"></a>Мониторинг задания резервного копирования

Чтобы отслеживать состояние заданий резервного копирования, используйте команду [az backup job list](/cli/azure/backup/job#az_backup_job_list):

```azurecli-interactive
az backup job list \
    --resource-group myResourceGroup \
    --vault-name myRecoveryServicesVault \
    --output table
```

В результате вы получите приблизительно следующие выходные данные. Они указывают на то, что задание резервного копирования выполняется (*InProgress*):

```output
Name      Operation        Status      Item Name    Start Time UTC       Duration
--------  ---------------  ----------  -----------  -------------------  --------------
a0a8e5e6  Backup           InProgress  myvm         2017-09-19T03:09:21  0:00:48.718366
fe5d0414  ConfigureBackup  Completed   myvm         2017-09-19T03:03:57  0:00:31.191807
```

Если в столбце *Status* (Состояние) задания резервного копирования отображается значение *Completed* (Выполнено), это означает, что виртуальная машина защищена с помощью служб восстановления и для нее сохранена точка полного восстановления.

## <a name="clean-up-deployment"></a>Очистка развертывания

Если защита больше не нужна, вы можете отключить ее на виртуальной машине, удалить точки восстановления и хранилище служб восстановления, а затем удалить группу ресурсов и связанные ресурсы виртуальной машины. Если вы использовали существующую виртуальную машину, можете пропустить последнюю команду [az group delete](/cli/azure/group#az_group_delete), чтобы оставить группу ресурсов и виртуальную машину.

Если вы хотите ознакомиться с руководством по резервному копированию, в котором объясняется, как восстанавливать данные виртуальной машины, перейдите к [дальнейшим действиям](#next-steps).

```azurecli-interactive
az backup protection disable \
    --resource-group myResourceGroup \
    --vault-name myRecoveryServicesVault \
    --container-name myVM \
    --item-name myVM \
    --delete-backup-data true
az backup vault delete \
    --resource-group myResourceGroup \
    --name myRecoveryServicesVault \
az group delete --name myResourceGroup
```

## <a name="next-steps"></a>Дальнейшие действия

В этом кратком руководстве вы создали хранилище служб восстановления, включили защиту на виртуальной машине и создали начальную точку восстановления. Дополнительные сведения о службе Azure Backup и службах восстановления см. в соответствующих руководствах.

> [!div class="nextstepaction"]
> [Резервное копирование нескольких виртуальных машин Azure](./tutorial-backup-vm-at-scale.md)
