---
title: Включение управления версиями BLOB-объектов и работа с ним
titleSuffix: Azure Storage
description: Узнайте, как включить управление версиями BLOB-объектов в портал Azure или с помощью шаблона Azure Resource Manager.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 02/09/2021
ms.author: tamram
ms.subservice: blobs
ms.custom: devx-track-csharp
ms.openlocfilehash: 5b6bd16eacf4b1bbb7b93f5500813e7fa9dc7eef
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100095853"
---
# <a name="enable-and-manage-blob-versioning"></a>Включение управления версиями BLOB-объектов и работа с ним

Вы можете включить управление версиями хранилища BLOB-объектов, чтобы автоматически поддерживать предыдущие версии большого двоичного объекта при его изменении или удалении. При включении управления версиями BLOB-объектов можно восстановить более раннюю версию большого двоичного объекта, чтобы восстановить данные, если они ошибочно изменены или удалены.

В этой статье показано, как включить или отключить управление версиями BLOB-объектов для учетной записи хранения с помощью портал Azure или шаблона Azure Resource Manager. Дополнительные сведения о управлении версиями BLOB-объектов см. в разделе [Управление версиями BLOB](versioning-overview.md)-объектов.

[!INCLUDE [storage-data-lake-gen2-support](../../../includes/storage-data-lake-gen2-support.md)]

## <a name="enable-blob-versioning"></a>Включение управления версиями BLOB-объектов

# <a name="azure-portal"></a>[Портал Azure](#tab/portal)

Чтобы включить управление версиями BLOB-объектов для учетной записи хранения в портал Azure:

1. Перейдите к своей учетной записи хранения на портале.
1. В разделе **Служба BLOB-объектов** выберите **Защита данных**.
1. В разделе **Управление версиями** выберите **включено**.

:::image type="content" source="media/versioning-enable/portal-enable-versioning.png" alt-text="Снимок экрана, показывающий, как включить управление версиями BLOB-объектов в портал Azure":::

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Чтобы включить управление версиями BLOB-объектов для учетной записи хранения с помощью PowerShell, сначала установите модуль [AZ. Storage](https://www.powershellgallery.com/packages/Az.Storage) 2.3.0 или более поздней версии. Затем вызовите команду [Update-азсторажеблобсервицепроперти](/powershell/module/az.storage/update-azstorageblobserviceproperty) , чтобы включить управление версиями, как показано в следующем примере. Не забудьте заменить значения в угловых скобках собственными значениями:

```powershell
# Set resource group and account variables.
$rgName = "<resource-group>"
$accountName = "<storage-account>"

# Enable versioning.
Update-AzStorageBlobServiceProperty -ResourceGroupName $rgName `
    -StorageAccountName $accountName `
    -IsVersioningEnabled $true
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Чтобы включить управление версиями BLOB-объектов для учетной записи хранения с Azure CLI, сначала установите Azure CLI версии 2.2.0 или более поздней. Затем вызовите команду [AZ Storage Account BLOB-Service-Properties Update](/cli/azure/ext/storage-blob-preview/storage/account/blob-service-properties#ext_storage_blob_preview_az_storage_account_blob_service_properties_update) , чтобы включить управление версиями, как показано в следующем примере. Не забудьте заменить значения в угловых скобках собственными значениями:

```azurecli
az storage account blob-service-properties update \
    --resource-group <resource_group> \
    --account-name <storage-account> \
    --enable-versioning true
```

# <a name="template"></a>[Шаблон](#tab/template)

Чтобы включить управление версиями BLOB-объектов с помощью шаблона, создайте шаблон со свойством **исверсионинженаблед** в **значение true**. Следующие шаги описывают создание шаблона в портал Azure.

1. В портал Azure выберите **создать ресурс**.
1. В **поле Поиск в Marketplace** введите **шаблон развертывание** и нажмите клавишу **Ввод**.
1. Выберите **шаблоны развертывания**, выберите **создать**, а затем — создать **собственный шаблон в редакторе**.
1. В редакторе шаблонов вставьте следующий код JSON. Замените заполнитель `<accountName>` именем вашей учетной записи хранения.
1. при сохранении шаблона;
1. Укажите группу ресурсов учетной записи, а затем нажмите кнопку **приобрести** , чтобы развернуть шаблон и включить управление версиями BLOB-объектов.

    ```json
    {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {},
        "variables": {},
        "resources": [
            {
                "type": "Microsoft.Storage/storageAccounts/blobServices",
                "apiVersion": "2019-06-01",
                "name": "<accountName>/default",
                "properties": {
                    "IsVersioningEnabled": true
                }
            }
        ]
    }
    ```

Дополнительные сведения о развертывании ресурсов с помощью шаблонов в портал Azure см. в разделе [развертывание ресурсов с помощью портал Azure](../../azure-resource-manager/templates/deploy-portal.md).

---

## <a name="modify-a-blob-to-trigger-a-new-version"></a>Изменение большого двоичного объекта для активации новой версии

В следующем примере кода показано, как активировать создание новой версии с помощью клиентской библиотеки службы хранилища Azure для .NET версии [12.5.1](https://www.nuget.org/packages/Azure.Storage.Blobs/12.5.1) или более поздней. Перед запуском этого примера убедитесь, что вы включили управление версиями для учетной записи хранения.

В примере создается блочный BLOB-объект, а затем обновляются метаданные большого двоичного объекта. Обновление метаданных большого двоичного объекта инициирует создание новой версии. В примере извлекается исходная и текущая версии, и показывается, что метаданные содержат только текущая версия.

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/CRUD.cs" id="Snippet_UpdateVersionedBlobMetadata":::

## <a name="list-blob-versions"></a>Вывод списка версий BLOB-объектов

Чтобы получить список версий BLOB-объектов или моментальных снимков с клиентской библиотекой .NET 12, укажите параметр [блобстатес](/dotnet/api/azure.storage.blobs.models.blobstates) в поле **Version** .

В следующем примере кода показано, как вывести версию BLOB-объектов с помощью клиентской библиотеки службы хранилища Azure для .NET версии [12.5.1](https://www.nuget.org/packages/Azure.Storage.Blobs/12.5.1) или более поздней. Перед запуском этого примера убедитесь, что вы включили управление версиями для учетной записи хранения.

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/CRUD.cs" id="Snippet_ListBlobVersions":::

## <a name="next-steps"></a>Дальнейшие действия

- [Управление версиями BLOB-объектов](versioning-overview.md)
- [Soft delete for Azure Storage blobs](./soft-delete-blob-overview.md) (Обратимое удаление больших двоичных объектов службы хранилища Azure)