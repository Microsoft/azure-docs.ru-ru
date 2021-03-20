---
title: Включение и контроль обратимого удаления для BLOB-объектов
titleSuffix: Azure Storage
description: Включите обратимое удаление для больших двоичных объектов, чтобы упростить восстановление данных при ошибочном изменении или удалении.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 07/15/2020
ms.author: tamram
ms.subservice: blobs
ms.custom: devx-track-azurecli, devx-track-csharp
ms.openlocfilehash: a74860b7adf4dade5aedc71a4960595cbe55eaf0
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "95995307"
---
# <a name="enable-and-manage-soft-delete-for-blobs"></a>Включение и контроль обратимого удаления для BLOB-объектов

Обратимое удаление BLOB-объектов защищает данные от случайного или ошибочного изменения или удаления. Если обратимое удаление BLOB-объектов включено для учетной записи хранения, большие двоичные объекты, версии больших двоичных объектов и моментальные снимки в этой учетной записи хранения могут быть восстановлены после удаления в течение указанного срока хранения.

Если есть вероятность, что данные могут быть случайно изменены или удалены приложением или другим пользователем учетной записи хранения, корпорация Майкрософт рекомендует включить обратимое удаление BLOB-объектов. В этой статье показано, как включить обратимое удаление для больших двоичных объектов. Дополнительные сведения об обратимом удалении BLOB-объектов см. в статье [обратимое удаление для больших двоичных объектов](soft-delete-blob-overview.md).

Сведения о включении обратимого удаления для контейнеров см. в разделе [Включение обратимого удаления для контейнеров и управление ими](soft-delete-container-enable.md).

## <a name="enable-blob-soft-delete"></a>Включение обратимого удаления BLOB-объектов

# <a name="portal"></a>[Портал](#tab/azure-portal)

Включите обратимое удаление для BLOB-объектов в учетной записи хранения с помощью портала Azure:

1. Войдите в свою учетную запись хранения на [портале Azure](https://portal.azure.com/).
1. Выберите параметр **Защита данных** в разделе **Служба BLOB-объектов**.
1. Задайте для свойства " **обратимое удаление больших двоичных объектов** " значение *включено*.
1. В разделе **политики хранения** укажите, как долго удаляемые обратимо удаленные BLOB-объекты будут храниться в службе хранилища Azure.
1. Сохраните изменения.

![Снимок экрана портала Azure с выбранной службой BLOB-объектов защиты данных.](media/soft-delete-blob-enable/storage-blob-soft-delete-portal-configuration.png)

Чтобы просмотреть обратимо удаленные большие двоичные объекты, установите флажок **Show deleted blobs** (Показать удаленные большие двоичные объекты).

![Снимок экрана со страницей службы BLOB-объектов для защиты данных с выделенным параметром "Показывать удаленные большие двоичные объекты".](media/soft-delete-blob-enable/storage-blob-soft-delete-portal-view-soft-deleted.png)

Чтобы просмотреть обратимо удаленные моментальные снимки для данного большого двоичного объекта, выберите большой двоичный объект, а затем щелкните **Просмотреть моментальные снимки**.

![Снимок экрана со страницей службы BLOB-объектов для защиты данных с выделенным параметром "просмотреть моментальные снимки".](media/soft-delete-blob-enable/storage-blob-soft-delete-portal-view-soft-deleted-snapshots.png)

Установите флажок **Показать удаленные моментальные снимки**.

![Снимок экрана страницы "Просмотр моментальных снимков" с выделенным параметром "Показать удаленные BLOB-объекты".](media/soft-delete-blob-enable/storage-blob-soft-delete-portal-view-soft-deleted-snapshots-check.png)

Щелкнув обратимо удаленный большой двоичный объект или моментальный снимок, обратите внимание на новые свойства большого двоичного объекта. Они указывают, когда объект был удален и количество дней, оставшихся до полного удаления большого двоичного объекта или моментального снимка. Если обратимо удаленный объект не является моментальным снимком, его удаление можно отменить.

![Снимок экрана с подробными сведениями об обратимо удаленном объекте.](media/soft-delete-blob-enable/storage-blob-soft-delete-portal-properties.png)

Помните, что отмена удаления большого двоичного объекта также отменит удаление всех связанных с ним моментальных снимков. Чтобы обратимо удалить моментальные снимки активного большого двоичного объекта, щелкните его и выберите **Undelete all snapshots** (Отменить удаление всех моментальных снимков).

![Снимок экрана со сведениями об обратимо удаленном большом двоичном объекте.](media/soft-delete-blob-enable/storage-blob-soft-delete-portal-undelete-all-snapshots.png)

После отмены удаления моментальных снимков большого двоичного объекта можно щелкнуть **Повысить уровень**, чтобы скопировать моментальный снимок в корневой большой двоичный объект, тем самым восстановив большой двоичный объект в моментальный снимок.

![Снимок экрана страницы "Просмотр моментальных снимков" с выделенным параметром "повысить уровень".](media/soft-delete-blob-enable/storage-blob-soft-delete-portal-promote-snapshot.png)

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

Чтобы включить обратимое удаление, обновите свойства клиентской службы BLOB-объектов. В следующем примере включено обратимое удаление подмножества учетных записей в подписке:

```powershell
Set-AzContext -Subscription "<subscription-name>"
$MatchingAccounts = Get-AzStorageAccount | where-object{$_.StorageAccountName -match "<matching-regex>"}
$MatchingAccounts | Enable-AzStorageDeleteRetentionPolicy -RetentionDays 7
```

Убедиться, что обратимое удаление включено, можно с помощью следующей команды:

```powershell
$MatchingAccounts | $account = Get-AzStorageAccount -ResourceGroupName myresourcegroup -Name storageaccount
   Get-AzStorageServiceProperty -ServiceType Blob -Context $account.Context | Select-Object -ExpandProperty DeleteRetentionPolicy
```

Чтобы восстановить большие двоичные объекты, которые были случайно удалены, вызовите команду отмены **удаления больших двоичных** объектов. Помните, что при вызове операции **Undelete Blob** как в активных, так и в обратимо удаленных больших двоичных объектах все связанные обратимо удаленные моментальные снимки будут восстановлены в качестве активных. В следующем примере вызывается **Удаление большого двоичного объекта** во всех обратимо удаленных и активных больших двоичных объектах в контейнере:

```powershell
# Create a context by specifying storage account name and key
$ctx = New-AzStorageContext -StorageAccountName $StorageAccountName -StorageAccountKey $StorageAccountKey

# Get the blobs in a given container and show their properties
$Blobs = Get-AzStorageBlob -Container $StorageContainerName -Context $ctx -IncludeDeleted
$Blobs.ICloudBlob.Properties

# Undelete the blobs
$Blobs.ICloudBlob.Undelete()
```
Чтобы найти текущую политику хранения обратимого удаления, используйте следующую команду:

```azurepowershell-interactive
   $account = Get-AzStorageAccount -ResourceGroupName myresourcegroup -Name storageaccount
   Get-AzStorageServiceProperty -ServiceType Blob -Context $account.Context
```

# <a name="cli"></a>[CLI](#tab/azure-CLI)

Чтобы включить обратимое удаление, обновите свойства клиентской службы BLOB-объектов:

```azurecli-interactive
az storage blob service-properties delete-policy update --days-retained 7  --account-name mystorageaccount --enable true
```

Чтобы проверить, включена ли возможность удаления, используйте следующую команду: 

```azurecli-interactive
az storage blob service-properties delete-policy show --account-name mystorageaccount 
```

# <a name="python"></a>[Python](#tab/python)

Чтобы включить обратимое удаление, обновите свойства клиентской службы BLOB-объектов:

```python
# Make the requisite imports
from azure.storage.blob import BlockBlobService
from azure.storage.common.models import DeleteRetentionPolicy

# Initialize a block blob service
block_blob_service = BlockBlobService(
    account_name='<enter your storage account name>', account_key='<enter your storage account key>')

# Set the blob client's service property settings to enable soft delete
block_blob_service.set_blob_service_properties(
    delete_retention_policy=DeleteRetentionPolicy(enabled=True, days=7))
```

# <a name="net-v12"></a>[.NET (версии 12)](#tab/dotnet)

Чтобы включить обратимое удаление, обновите свойства клиентской службы BLOB-объектов:

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/DataProtection.cs" id="Snippet_EnableSoftDelete":::

Чтобы восстановить большие двоичные объекты, которые были случайно удалены, вызовите для них отмену удаления. Помните, что при вызове операции **Отменить удаление** как в активных, так и в обратимо удаленных BLOB-объектах все связанные обратимо удаленные моментальные снимки будут восстановлены в качестве активных. В следующем примере вызывается отмена удаления всех обратимо удаленных и активных больших двоичных объектов в контейнере:

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/DataProtection.cs" id="Snippet_RecoverDeletedBlobs":::

Чтобы выполнить восстановление до определенной версии большого двоичного объекта, сначала необходимо вызвать отмену удаления большого двоичного объекта, а затем скопировать требуемый моментальный снимок в большой двоичный объект. В следующем примере блочный BLOB-объект восстанавливается до недавно созданного моментального снимка:

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/DataProtection.cs" id="Snippet_RecoverSpecificBlobVersion":::

# <a name="net-v11"></a>[.NET (версии 11)](#tab/dotnet11)

Чтобы включить обратимое удаление, обновите свойства клиентской службы BLOB-объектов:

```csharp
// Get the blob client's service property settings
ServiceProperties serviceProperties = blobClient.GetServiceProperties();

// Configure soft delete
serviceProperties.DeleteRetentionPolicy.Enabled = true;
serviceProperties.DeleteRetentionPolicy.RetentionDays = RetentionDays;

// Set the blob client's service property settings
blobClient.SetServiceProperties(serviceProperties);
```

Чтобы восстановить большие двоичные объекты, которые были случайно удалены, вызовите команду отмены **удаления больших двоичных** объектов. Помните, что при вызове операции **Undelete Blob** как в активных, так и в обратимо удаленных больших двоичных объектах все связанные обратимо удаленные моментальные снимки будут восстановлены в качестве активных. В следующем примере вызывается **Удаление большого двоичного объекта** для всех обратимо удаленных и активных больших двоичных объектов в контейнере:

```csharp
// Recover all blobs in a container
foreach (CloudBlob blob in container.ListBlobs(useFlatBlobListing: true, blobListingDetails: BlobListingDetails.Deleted))
{
       await blob.UndeleteAsync();
}
```

Чтобы выполнить восстановление до определенной версии большого двоичного объекта, сначала вызовите операцию **отмены удаления большого двоичного объекта** , а затем скопируйте нужный моментальный снимок на большой двоичный объект. В следующем примере блочный BLOB-объект восстанавливается до недавно созданного моментального снимка:

```csharp
// Undelete
await blockBlob.UndeleteAsync();

// List all blobs and snapshots in the container prefixed by the blob name
IEnumerable<IListBlobItem> allBlobVersions = container.ListBlobs(
    prefix: blockBlob.Name, useFlatBlobListing: true, blobListingDetails: BlobListingDetails.Snapshots);

// Restore the most recently generated snapshot to the active blob
CloudBlockBlob copySource = allBlobVersions.First(version => ((CloudBlockBlob)version).IsSnapshot &&
    ((CloudBlockBlob)version).Name == blockBlob.Name) as CloudBlockBlob;
blockBlob.StartCopy(copySource);
```  

---

## <a name="next-steps"></a>Дальнейшие действия

- [Обратимое удаление для хранилища BLOB-объектов](./soft-delete-blob-overview.md)
- [Управление версиями BLOB-объектов](versioning-overview.md)