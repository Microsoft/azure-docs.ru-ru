---
title: Безопасный доступ к данным приложения
titleSuffix: Azure Storage
description: Защита данных приложения в облаке с помощью маркеров SAS, шифрования и протокола HTTPS.
services: storage
author: tamram
ms.service: storage
ms.subservice: blobs
ms.topic: tutorial
ms.date: 06/10/2020
ms.author: tamram
ms.reviewer: ozgun
ms.custom: mvc, devx-track-csharp, devx-track-azurecli
ms.openlocfilehash: f7c5dbaf30965fdd5f438f0351cfa2cd60e05b70
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92746551"
---
# <a name="secure-access-to-application-data"></a>Безопасный доступ к данным приложения

Это руководство представляет собой первую часть цикла. Вы узнаете, как обеспечить безопасность доступа к учетной записи хранения. 

В третьей части цикла вы узнаете, как выполнять такие задачи:

> [!div class="checklist"]
> * Использование маркеров SAS для доступа к эскизам изображений
> * Включение шифрования на стороне сервера
> * Включение транспорта только по протоколу HTTPS

[Хранилище BLOB-объектов](../common/storage-introduction.md#blob-storage) представляет собой надежную службу для хранения файлов для приложений. Это руководство дополняет сведения из [предыдущего раздела][previous-tutorial] и описывает процедуру защиты доступа к учетной записи хранения из веб-приложения. По окончании работы изображения шифруются, и для доступа к эскизам веб-приложение использует маркеры безопасности SAS.

## <a name="prerequisites"></a>Предварительные требования

Для работы с этим руководством необходимо изучить предыдущее руководство по использованию хранилища — [Автоматическое изменение размера переданных изображений с помощью Сетки событий][previous-tutorial].

## <a name="set-container-public-access"></a>Установка общего доступа к контейнеру

В этой части серии учебников для доступа к эскизам используются маркеры SAS. На этом шаге вы установите `off` в качестве значения общего доступа к контейнеру *эскизов*.

```bash
blobStorageAccount="<blob_storage_account>"

blobStorageAccountKey=$(az storage account keys list -g myResourceGroup \
    --account-name $blobStorageAccount --query [0].value --output tsv) 

az storage container set-permission \
    --account-name $blobStorageAccount \
    --account-key $blobStorageAccountKey \
    --name thumbnails \
    --public-access off
```

```powershell
$blobStorageAccount="<blob_storage_account>"

blobStorageAccountKey=$(az storage account keys list -g myResourceGroup `
    --account-name $blobStorageAccount --query [0].value --output tsv) 

az storage container set-permission `
    --account-name $blobStorageAccount `
    --account-key $blobStorageAccountKey `
    --name thumbnails `
    --public-access off
```

## <a name="configure-sas-tokens-for-thumbnails"></a>Настройка маркеров SAS для эскизов

В первой части этой серии учебников в веб-приложении отображались изображения из общедоступного контейнера. В этой части вы будете использовать маркеры SAS для получения изображений эскизов. Маркеры SAS позволяют предоставлять ограниченный доступ к контейнеру или BLOB-объекту на основе IP-адреса, протокола, интервала времени или имеющихся прав. Дополнительные сведения о подписанных URL-адресах см. в статье об [использование подписанных URL-адресов SAS в службе хранилища Azure](../common/storage-sas-overview.md).

В этом примере репозиторий исходного кода использует ветвь `sasTokens` с примером обновленного кода. Удалите существующее развертывание GitHub с помощью команды [az webapp deployment source delete](/cli/azure/webapp/deployment/source). Затем настройте для веб-приложения развертывание GitHub с помощью команды [az webapp deployment source config](/cli/azure/webapp/deployment/source).

В следующей команде `<web-app>` — это имя веб-приложения.

```bash
az webapp deployment source delete --name <web-app> --resource-group myResourceGroup

az webapp deployment source config --name <web_app> \
    --resource-group myResourceGroup --branch sasTokens --manual-integration \
    --repo-url https://github.com/Azure-Samples/storage-blob-upload-from-webapp
```

```powershell
az webapp deployment source delete --name <web-app> --resource-group myResourceGroup

az webapp deployment source config --name <web_app> `
    --resource-group myResourceGroup --branch sasTokens --manual-integration `
    --repo-url https://github.com/Azure-Samples/storage-blob-upload-from-webapp
```

Ветвь `sasTokens` репозитория обновляет файл `StorageHelper.cs`. Она заменяет задачу `GetThumbNailUrls` на пример кода ниже. Обновленная задача получает URL-адреса эскиза с помощью класса [BlobSasBuilder](/dotnet/api/azure.storage.sas.blobsasbuilder), чтобы указать время начала, время окончания и разрешения для маркера SAS. Развернутое веб-приложение получает эскизы с URL-адресом с помощью токена SAS. В следующем примере показана обновленная задача.

```csharp
public static async Task<List<string>> GetThumbNailUrls(AzureStorageConfig _storageConfig)
{
    List<string> thumbnailUrls = new List<string>();

    // Create a URI to the storage account
    Uri accountUri = new Uri("https://" + _storageConfig.AccountName + ".blob.core.windows.net/");

    // Create BlobServiceClient from the account URI
    BlobServiceClient blobServiceClient = new BlobServiceClient(accountUri);

    // Get reference to the container
    BlobContainerClient container = blobServiceClient.GetBlobContainerClient(_storageConfig.ThumbnailContainer);

    if (container.Exists())
    {
        // Set the expiration time and permissions for the container.
        // In this case, the start time is specified as a few 
        // minutes in the past, to mitigate clock skew.
        // The shared access signature will be valid immediately.
        BlobSasBuilder sas = new BlobSasBuilder
        {
            Resource = "c",
            BlobContainerName = _storageConfig.ThumbnailContainer,
            StartsOn = DateTimeOffset.UtcNow.AddMinutes(-5),
            ExpiresOn = DateTimeOffset.UtcNow.AddHours(1)
        };

        sas.SetPermissions(BlobContainerSasPermissions.All);

        // Create StorageSharedKeyCredentials object by reading
        // the values from the configuration (appsettings.json)
        StorageSharedKeyCredential storageCredential =
            new StorageSharedKeyCredential(_storageConfig.AccountName, _storageConfig.AccountKey);

        // Create a SAS URI to the storage account
        UriBuilder sasUri = new UriBuilder(accountUri);
        sasUri.Query = sas.ToSasQueryParameters(storageCredential).ToString();

        foreach (BlobItem blob in container.GetBlobs())
        {
            // Create the URI using the SAS query token.
            string sasBlobUri = container.Uri + "/" +
                                blob.Name + sasUri.Query;

            //Return the URI string for the container, including the SAS token.
            thumbnailUrls.Add(sasBlobUri);
        }
    }
    return await Task.FromResult(thumbnailUrls);
}
```

В предыдущей задаче используются следующие классы, свойства и методы:

| Class | Свойства | Методы |
|-------|------------|---------|
|[StorageSharedKeyCredential](/dotnet/api/azure.storage.storagesharedkeycredential) |  |  |
|[BlobServiceClient](/dotnet/api/azure.storage.blobs.blobserviceclient) |  |[GetBlobContainerClient](/dotnet/api/azure.storage.blobs.blobserviceclient.getblobcontainerclient) |
|[BlobContainerClient](/dotnet/api/azure.storage.blobs.blobcontainerclient) | [URI](/dotnet/api/azure.storage.blobs.blobcontainerclient.uri) |[Exists](/dotnet/api/azure.storage.blobs.blobcontainerclient.exists) <br> [GetBlobs](/dotnet/api/azure.storage.blobs.blobcontainerclient.getblobs) |
|[BlobSasBuilder](/dotnet/api/azure.storage.sas.blobsasbuilder) |  | [SetPermissions](/dotnet/api/azure.storage.sas.blobsasbuilder.setpermissions) <br> [ToSasQueryParameters](/dotnet/api/azure.storage.sas.blobsasbuilder.tosasqueryparameters) |
|[BlobItem](/dotnet/api/azure.storage.blobs.models.blobitem) | [имя](/dotnet/api/azure.storage.blobs.models.blobitem.name); |  |
|[UriBuilder](/dotnet/api/system.uribuilder) | [Запрос](/dotnet/api/system.uribuilder.query) |  |
|[Список](/dotnet/api/system.collections.generic.list-1) | | [Добавление](/dotnet/api/system.collections.generic.list-1.add) |

## <a name="azure-storage-encryption"></a>Шифрование службы хранилища Azure

[Шифрование службы хранилища Azure](../common/storage-service-encryption.md) помогает защитить данные путем шифрования неактивных данных и управления шифрованием и расшифровкой. Используется 256-битное [шифрование AES](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard), самая надежная технология блочного шифрования на сегодняшний день.

Вы можете делегировать корпорации Майкрософт управление ключами шифрования или использовать собственные ключи с ключами, управляемыми клиентами, хранящимися в Azure Key Vault или на устройстве HSM (Hardware Security Model), управляемом Key Vault (предварительная версия). Дополнительные сведения см. в статье [Ключи, управляемые клиентом, для шифрования службы хранилища Azure](../common/customer-managed-keys-overview.md).

Шифрование службы хранилища Azure автоматически шифрует данные на всех уровнях производительности ("Стандартный" и "Премиум"), во всех моделях развертывания (Azure Resource Manager и классической) и во всех службах хранилища Azure (больших двоичных объектов, очередей, таблиц и файлов).

## <a name="enable-https-only"></a>Включение только HTTPS

Чтобы обеспечить безопасность запросов данных в учетную запись хранения и из нее, можно разрешить только запросы HTTPS. Обновите необходимый протокол для учетной записи хранения с помощью команды [az storage account update](/cli/azure/storage/account).

```azurecli-interactive
az storage account update --resource-group myresourcegroup --name <storage-account-name> --https-only true
```

Протестируйте подключение, использующее `curl`, с помощью протокола `HTTP`.

```azurecli-interactive
curl http://<storage-account-name>.blob.core.windows.net/<container>/<blob-name> -I
```

Теперь, когда требуется безопасная передача, появляется следующее сообщение:

```
HTTP/1.1 400 The account being accessed does not support http.
```

## <a name="next-steps"></a>Дальнейшие действия

В третьей части серии вы узнали, как обеспечить безопасность доступа к учетной записи хранения путем выполнения приведенных ниже задач.

> [!div class="checklist"]
> * Использование маркеров SAS для доступа к эскизам изображений
> * Включение шифрования на стороне сервера
> * Включение транспорта только по протоколу HTTPS

Перейдите к четвертой части, чтобы узнать, как отслеживать приложение облачного хранилища и устранять его неполадки.

> [!div class="nextstepaction"]
> [Мониторинг и устранение неполадок приложения облачного хранилища](storage-monitor-troubleshoot-storage-application.md)

[previous-tutorial]: ../../event-grid/resize-images-on-storage-blob-upload-event.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json
