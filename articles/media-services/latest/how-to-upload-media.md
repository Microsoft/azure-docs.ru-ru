---
заголовок: Отправка мультимедиа : Службы мультимедиа Azure description: Узнайте, как отправить мультимедиа для потоковой передачи или кодирования.
services: media-services documentationcenter: '' author: IngridAtMicrosoft manager: femila editor: ''

ms.service: media-services ms.workload: ms.topic: how-to ms.date: 08/31/2020 ms.author: inhenkel
---

# <a name="upload-media-for-streaming-or-encoding"></a>Отправка мультимедиа для потоковой передачи или кодирования.

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

В Службах мультимедиа цифровые файлы (мультимедиа) отправляются в контейнер BLOB-объектов, связанный с ресурсом. Сущность [Ресурс](/rest/api/media/operations/asset) может содержать видео, аудио, изображения, коллекции эскизов, текстовые каналы и файлы скрытых субтитров (а также метаданные этих файлов). После отправки этих файлов в контейнер ресурса содержимое сохраняется в безопасном расположении в облаке для дальнейшей обработки и потоковой передачи.

Прежде чем приступать к работе, необходимо определить следующее:

1. Локальный путь к файлу, который необходимо передать.
1. Идентификатор ресурса (контейнера).
1. Имя, которое будет использоваться для отправленного файла, включая расширение.
1. Имя используемой учетной записи хранения.
1. Ключ доступа для вашей учетной записи хранения.

## <a name="portal"></a>[Портал](#tab/portal/)

[!INCLUDE [Upload files with the portal](./includes/task-upload-file-to-asset-portal.md)]

## <a name="cli"></a>[CLI](#tab/cli/)

[!INCLUDE [Upload files with the CLI](./includes/task-upload-file-to-asset-cli.md)]

## <a name="python"></a>[Python](#tab/python)

Если в коде уже настроен способ аутентификации и вы уже создали ресурс ввода, используйте следующий фрагмент кода для передачи локальных файлов в этот ресурс (in_container).

```python
#The storage objects
from azure.storage.blob import BlobServiceClient, BlobClient

#Establish storage variables
storage_account_name = '<your storage account name'
storage_account_key = '<your storage account key'
storage_blob_url = 'https://<your storage account name>.blob.core.windows.net/'

in_container = 'asset-' + inputAsset.asset_id

#The file path of local file you want to upload
source_file = "ignite.mp4"

# Use the Storage SDK to upload the video
blob_service_client = BlobServiceClient(account_url=storage_blob_url, credential=storage_account_key)
blob_client = blob_service_client.get_blob_client(in_container,source_file)

# Upload the video to storage as a block blob
with open(source_file, "rb") as data:
    blob_client.upload_blob(data, blob_type="BlockBlob")
```

---
<!-- add these to the tabs when available -->
Другие методы см. в [документации по службе хранилища Azure](../../storage/blobs/index.yml) для работы с BLOB-объектами в [.NET](../../storage/blobs/storage-quickstart-blobs-dotnet.md), [Java](../../storage/blobs/storage-quickstart-blobs-java.md), [Python](../../storage/blobs/storage-quickstart-blobs-python.md)и [JavaScript (Node.js)](../../storage/blobs/storage-quickstart-blobs-nodejs.md).

## <a name="next-steps"></a>Дальнейшие действия

> [Общие сведения о службах мультимедиа](media-services-overview.md)
