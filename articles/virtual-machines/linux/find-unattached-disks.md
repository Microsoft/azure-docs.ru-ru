---
title: Azure CLI — Поиск и удаление неподключенных управляемых и неуправляемых дисков
description: Как найти и удалить неподключенные управляемые и неуправляемые диски Azure (виртуальные жесткие диски и страничные BLOB-объекты) с помощью Azure CLI.
author: roygara
ms.service: virtual-machines
ms.topic: how-to
ms.date: 03/30/2018
ms.author: rogarana
ms.subservice: disks
ms.openlocfilehash: 0c3e8bb2ff6f3313e851a4253a95a5ad923a8f70
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96016222"
---
# <a name="find-and-delete-unattached-azure-managed-and-unmanaged-disks-using-the-azure-cli"></a>Поиск и удаление неподключенных управляемых и неуправляемых дисков Azure с помощью Azure CLI
Когда вы удаляете виртуальную машину в Azure, подключенные к ней диски не удаляются по умолчанию. Это предотвращает потерю данных из-за случайного удаления виртуальной машины. Тем не менее вы продолжаете платить за неподключенные диски. В этой статье показано, как найти и удалить все неподключенные диски и сократить затраты. 


## <a name="managed-disks-find-and-delete-unattached-disks"></a>Управляемые диски. Поиск и удаление неподключенных дисков 

В следующем скрипте показано, как с помощью свойства **ManagedBy** найти неподключенные [управляемые диски](../managed-disks-overview.md). В свойстве **ManagedBy** хранится идентификатор ресурса виртуальной машины, к которой подключен управляемый диск. Если управляемый диск не подключен, свойство **ManagedBy** имеет значение NULL. Сценарий проверяет все управляемые диски в подписке Azure. Когда скрипт находит управляемый диск со значением свойства **ManagedBy** равным null, он определяет, что диск не подключен.

>[!IMPORTANT]
>Сначала запустите скрипт, установив для переменной **deleteUnattachedDisks** значение 0. Это действие позволяет найти и просмотреть все неподключенные управляемые диски.
>
>После просмотра всех неподключенных дисков запустите скрипт еще раз и установите для переменной **deleteUnattachedDisks** значение 1. Это действие позволяет удалить все неподключенные управляемые диски.
>

```azurecli
# Set deleteUnattachedDisks=1 if you want to delete unattached Managed Disks
# Set deleteUnattachedDisks=0 if you want to see the Id of the unattached Managed Disks
deleteUnattachedDisks=0
unattachedDiskIds=$(az disk list --query '[?managedBy==`null`].[id]' -o tsv)
for id in ${unattachedDiskIds[@]}
do
    if (( $deleteUnattachedDisks == 1 ))
    then

        echo "Deleting unattached Managed Disk with Id: "$id
        az disk delete --ids $id --yes
        echo "Deleted unattached Managed Disk with Id: "$id

    else
        echo $id
    fi
done
```

## <a name="unmanaged-disks-find-and-delete-unattached-disks"></a>Неуправляемые диски. Поиск и удаление неподключенных дисков 

Неуправляемые диски — это VHD-файлы, которые хранятся в [учетных записях хранения Azure](../../storage/common/storage-account-overview.md) как [страничные BLOB-объекты](/rest/api/storageservices/understanding-block-blobs--append-blobs--and-page-blobs#about-page-blobs). В следующем скрипте показано, как с помощью свойства **LeaseStatus** найти неподключенные неуправляемые диски (страничные BLOB-объекты). Если неуправляемый диск подключен к виртуальной машине, свойство **LeaseStatus** имеет значение **Locked**. Если неуправляемый диск не подключен к виртуальной машине, свойство **LeaseStatus** имеет значение **Unlocked**. Сценарий проверяет все неуправляемые диски во всех учетных записях хранения Azure в подписке Azure. Когда скрипт находит неуправляемый диск со значением свойства **LeaseStatus** равным **Unlocked**, он определяет, что диск не подключен.

>[!IMPORTANT]
>Сначала запустите скрипт, установив для переменной **deleteUnattachedVHDs** значение 0. Это действие позволяет найти и просмотреть все неподключенные неуправляемые виртуальные жесткие диски.
>
>После просмотра всех неподключенных дисков запустите скрипт еще раз и установите для переменной **deleteUnattachedVHDs** значение 1. Это действие позволяет удалить все неподключенные неуправляемые виртуальные жесткие диски.
>

```azurecli
# Set deleteUnattachedVHDs=1 if you want to delete unattached VHDs
# Set deleteUnattachedVHDs=0 if you want to see the details of the unattached VHDs
deleteUnattachedVHDs=0
storageAccountIds=$(az storage account list --query [].[id] -o tsv)
for id in ${storageAccountIds[@]}
do
    connectionString=$(az storage account show-connection-string --ids $id --query connectionString -o tsv)
    containers=$(az storage container list --connection-string $connectionString --query [].[name] -o tsv)

    for container in ${containers[@]}
    do 
        
        blobs=$(az storage blob list -c $container --connection-string $connectionString --query "[?properties.blobType=='PageBlob' && ends_with(name,'.vhd')].[name]" -o tsv)
        
        for blob in ${blobs[@]}
        do
            leaseStatus=$(az storage blob show -n $blob -c $container --connection-string $connectionString --query "properties.lease.status" -o tsv)
            
            if [ "$leaseStatus" == "unlocked" ]
            then 

                if (( $deleteUnattachedVHDs == 1 ))
                then 

                    echo "Deleting VHD: "$blob" in container: "$container" in storage account: "$id

                    az storage blob delete --delete-snapshots include  -n $blob -c $container --connection-string $connectionString

                    echo "Deleted VHD: "$blob" in container: "$container" in storage account: "$id
                else
                    echo "StorageAccountId: "$id" container: "$container" VHD: "$blob
                fi

            fi
        done
    done
done 
```

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения см. в разделе [Удаление учетной записи хранения](../../storage/common/storage-account-create.md#delete-a-storage-account).