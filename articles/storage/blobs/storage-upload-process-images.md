---
title: Передача данных изображений в облако с помощью службы хранилища Azure | Документация Майкрософт
description: Использование хранилища BLOB-объектов Azure с веб-приложениями для хранения данных приложений в учетной записи хранения. В этом учебнике рассказывается, как создать веб-приложение, которое хранит и показывает изображения из службы хранилища Azure.
author: mhopkins-msft
ms.service: storage
ms.subservice: blobs
ms.topic: tutorial
ms.date: 06/24/2020
ms.author: mhopkins
ms.reviewer: dineshm
ms.custom: devx-track-js, devx-track-csharp, devx-track-azurecli
ms.openlocfilehash: b50aadfa16ed95bacb5247187c15489a1b017d39
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98676596"
---
# <a name="tutorial-upload-image-data-in-the-cloud-with-azure-storage"></a>Руководство по Передача данных изображений в облако с помощью службы хранилища Azure

Это руководство представляет первую часть цикла. В этом руководстве описано, как развернуть веб-приложение. Веб-приложение использует клиентскую библиотеку Хранилища BLOB-объектов Azure для отправки изображений в учетную запись хранения. По окончании у вас будет веб-приложение, которое хранит и показывает изображения из службы хранилища Azure.

# <a name="net-v12"></a>[\..NET (версии 12)](#tab/dotnet)

![Приложение для изменения размера изображений на платформе .NET](media/storage-upload-process-images/figure2.png)

# <a name="javascript-v12"></a>[JavaScript версии 12](#tab/javascript)

![Приложение для изменения размера изображений на платформе JavaScript](media/storage-upload-process-images/upload-app-nodejs-thumb.png)

---

В первой части цикла вы узнаете, как выполнять такие задачи:

> [!div class="checklist"]

> * Создание учетной записи хранения
> * Создание контейнера и настройка разрешений.
> * Получение ключа доступа.
> * Развертывание веб-приложения в Azure.
> * Настройка параметров приложения
> * Взаимодействие с веб-приложением.

## <a name="prerequisites"></a>Предварительные требования

Для работы с этим руководством вам потребуется подписка Azure. Перед началом работы создайте [бесплатную учетную запись](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio).

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

Чтобы установить и использовать интерфейс командной строки локально, запустите Azure CLI 2.0.4 или более поздней версии. Чтобы узнать версию, выполните команду `az --version`. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI](/cli/azure/install-azure-cli). 

## <a name="create-a-resource-group"></a>Создание группы ресурсов

Создайте группу ресурсов с помощью команды [az group create](/cli/azure/group). Группа ресурсов Azure является логическим контейнером, в котором происходит развертывание ресурсов Azure и управление ими.  

В следующем примере создается группа ресурсов `myResourceGroup`.

```azurecli
az group create --name myResourceGroup --location southeastasia
```

```powershell
az group create --name myResourceGroup --location southeastasia
```

## <a name="create-a-storage-account"></a>Создание учетной записи хранения

В примере изображения отправляются в контейнер больших двоичных объектов в учетной записи хранения Azure. Учетная запись хранения предоставляет уникальное пространство имен для хранения ваших объектов данных в службе хранилища Azure и доступа к ним. Создайте учетную запись хранения в созданной вами группе ресурсов с помощью команды [az storage account create](/cli/azure/storage/account).

> [!IMPORTANT]
> Во второй части этого руководства используется Сетка событий Azure для хранилища BLOB-объектов. Убедитесь в том, что созданная учетная запись хранения в регионе Azure поддерживает Сетку событий. Список поддерживаемых регионов см. в разделе [Доступность продуктов по регионам](https://azure.microsoft.com/global-infrastructure/services/?products=event-grid&regions=all).

В следующей команде замените глобально уникальное имя учетной записи хранения больших двоичных объектов там, где встречается заполнитель `<blob_storage_account>`.

```azurecli
blobStorageAccount="<blob_storage_account>"

az storage account create --name $blobStorageAccount --location southeastasia \
  --resource-group myResourceGroup --sku Standard_LRS --kind StorageV2 --access-tier hot
```

```powershell
$blobStorageAccount="<blob_storage_account>"

az storage account create --name $blobStorageAccount --location southeastasia `
  --resource-group myResourceGroup --sku Standard_LRS --kind StorageV2 --access-tier hot
```

## <a name="create-blob-storage-containers"></a>Создание контейнеров хранилища BLOB-объектов

Приложение использует два типа контейнеров в учетной записи хранилища больших двоичных объектов. Контейнеры похожи на папки и используются для хранения больших двоичных объектов. Контейнер *изображений* — это контейнер, в который отправляются изображения с высоким разрешением. В одной из последующих статей этой серии приложение-функция Azure будет отправлять эскизы изображений измененного размера в контейнер *эскизов*.

Получите ключи своей учетной записи хранения с помощью команды [az storage account keys list](/cli/azure/storage/account/keys). Затем используйте этот ключ для создания двух контейнеров с помощью команды [az storage container create](/cli/azure/storage/container).

Общему доступу к контейнеру *Изображения* присваивается значение `off`. Общему доступу к контейнеру *Эскизы* присваивается значение `container`. Параметр общего доступа `container` позволяет пользователям веб-страницы просматривать эскизы.

```bash
blobStorageAccountKey=$(az storage account keys list -g myResourceGroup \
  -n $blobStorageAccount --query "[0].value" --output tsv)

az storage container create --name images \
  --account-name $blobStorageAccount \
  --account-key $blobStorageAccountKey

az storage container create --name thumbnails \
  --account-name $blobStorageAccount \
  --account-key $blobStorageAccountKey --public-access container
```

```powershell
$blobStorageAccountKey=$(az storage account keys list -g myResourceGroup `
  -n $blobStorageAccount --query "[0].value" --output tsv)

az storage container create --name images `
  --account-name $blobStorageAccount `
  --account-key $blobStorageAccountKey

az storage container create --name thumbnails `
  --account-name $blobStorageAccount `
  --account-key $blobStorageAccountKey --public-access container
```

Запишите имя и ключ учетной записи хранения больших двоичных объектов. В этом примере приложения эти параметры используются для подключения к учетной записи хранения и отправки изображений. 

## <a name="create-an-app-service-plan"></a>Создание плана службы приложений

[План службы приложений](../../app-service/overview-hosting-plans.md) указывает расположение, размер и функции фермы веб-серверов, в которой размещается приложение.

Создайте план службы приложений, выполнив команду [az appservice plan create](/cli/azure/appservice/plan).

В следующем примере создается план службы приложений с именем `myAppServicePlan` и ценовой категорией **Бесплатный**.

```azurecli
az appservice plan create --name myAppServicePlan --resource-group myResourceGroup --sku Free
```

```powershell
az appservice plan create --name myAppServicePlan --resource-group myResourceGroup --sku Free
```

## <a name="create-a-web-app"></a>Создание веб-приложения

Веб-приложение предоставляет пространство для размещения примера кода приложения, развернутого из примера репозитория GitHub. Создайте [веб-приложение](../../app-service/overview.md) в рамках плана `myAppServicePlan` службы приложений с помощью команды [az webapp create](/cli/azure/webapp).  

В следующей команде замените `<web_app>` уникальным именем. Допустимые символы: `a-z`, `0-9` и `-`. Если `<web_app>` не является уникальным, отобразится сообщение об ошибке: *Веб-сайт с именем `<web_app>` уже существует.* URL-адрес приложения по умолчанию: `https://<web_app>.azurewebsites.net`.  

```azurecli
webapp="<web_app>"

az webapp create --name $webapp --resource-group myResourceGroup --plan myAppServicePlan
```

```powershell
$webapp="<web_app>"

az webapp create --name $webapp --resource-group myResourceGroup --plan myAppServicePlan
```

## <a name="deploy-the-sample-app-from-the-github-repository"></a>Развертывание примера приложения из репозитория GitHub

# <a name="net-v12"></a>[\..NET (версии 12)](#tab/dotnet)

Служба приложений поддерживает несколько способов развертывания содержимого для веб-приложения. В этом руководстве развертывается веб-приложение из [общедоступного репозитория примеров GitHub](https://github.com/Azure-Samples/storage-blob-upload-from-webapp). Настройте для веб-приложения развертывание GitHub с помощью команды [az webapp deployment source config](/cli/azure/webapp/deployment/source).

Пример проекта содержит приложение [ASP.NET MVC](https://www.asp.net/mvc). Оно принимает изображение, сохраняет его в учетную запись хранения и показывает изображения из контейнера эскизов. Веб-приложение использует пространства имен [Azure.Storage](/dotnet/api/azure.storage), [Azure.Storage.Blobs](/dotnet/api/azure.storage.blobs) и [Azure.Storage.Blobs.Models](/dotnet/api/azure.storage.blobs.models) для взаимодействия со службой хранилища Azure.

```azurecli
az webapp deployment source config --name $webapp --resource-group myResourceGroup \
  --branch master --manual-integration \
  --repo-url https://github.com/Azure-Samples/storage-blob-upload-from-webapp
```

```powershell
az webapp deployment source config --name $webapp --resource-group myResourceGroup `
  --branch master --manual-integration `
  --repo-url https://github.com/Azure-Samples/storage-blob-upload-from-webapp
```

# <a name="javascript-v12"></a>[JavaScript версии 12](#tab/javascript)

Служба приложений поддерживает несколько способов развертывания содержимого для веб-приложения. В этом руководстве развертывается веб-приложение из [общедоступного репозитория примеров GitHub](https://github.com/Azure-Samples/azure-sdk-for-js-storage-blob-stream-nodejs). Настройте для веб-приложения развертывание GitHub с помощью команды [az webapp deployment source config](/cli/azure/webapp/deployment/source).

```azurecli
az webapp deployment source config --name $webapp --resource-group myResourceGroup \
  --branch master --manual-integration \
  --repo-url https://github.com/Azure-Samples/azure-sdk-for-js-storage-blob-stream-nodejs
```

```powershell
az webapp deployment source config --name $webapp --resource-group myResourceGroup `
  --branch master --manual-integration `
  --repo-url https://github.com/Azure-Samples/azure-sdk-for-js-storage-blob-stream-nodejs
```

---

## <a name="configure-web-app-settings"></a>Настройка параметров веб-приложения

# <a name="net-v12"></a>[\..NET (версии 12)](#tab/dotnet)

В примере веб-приложения для отправки изображений используется [API службы хранилища Azure для .NET](/dotnet/api/overview/azure/storage). Учетные данные записи хранения задаются в параметрах приложения для веб-приложения. Добавить параметры в развернутое приложение можно с помощью команды [az webapp config appsettings set](/cli/azure/webapp/config/appsettings).

```azurecli
az webapp config appsettings set --name $webapp --resource-group myResourceGroup \
  --settings AzureStorageConfig__AccountName=$blobStorageAccount \
    AzureStorageConfig__ImageContainer=images \
    AzureStorageConfig__ThumbnailContainer=thumbnails \
    AzureStorageConfig__AccountKey=$blobStorageAccountKey
```

```powershell
az webapp config appsettings set --name $webapp --resource-group myResourceGroup `
  --settings AzureStorageConfig__AccountName=$blobStorageAccount `
    AzureStorageConfig__ImageContainer=images `
    AzureStorageConfig__ThumbnailContainer=thumbnails `
    AzureStorageConfig__AccountKey=$blobStorageAccountKey
```

# <a name="javascript-v12"></a>[JavaScript версии 12](#tab/javascript)

В примере веб-приложения для отправки изображений используется [клиентская библиотека службы хранилища Azure для JavaScript](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/storage). Учетные данные учетной записи хранения задаются в параметрах веб-приложения. Добавить параметры в развернутое приложение можно с помощью команды [az webapp config appsettings set](/cli/azure/webapp/config/appsettings).

```azurecli
az webapp config appsettings set --name $webapp --resource-group myResourceGroup \
  --settings AZURE_STORAGE_ACCOUNT_NAME=$blobStorageAccount \
    AZURE_STORAGE_ACCOUNT_ACCESS_KEY=$blobStorageAccountKey
```

```powershell
az webapp config appsettings set --name $webapp --resource-group myResourceGroup `
  --settings AZURE_STORAGE_ACCOUNT_NAME=$blobStorageAccount `
  AZURE_STORAGE_ACCOUNT_ACCESS_KEY=$blobStorageAccountKey
```

---

После развертывания и настройки веб-приложения можно проверить возможность отправки изображений в приложении.

## <a name="upload-an-image"></a>Передача образа

Чтобы проверить веб-приложение, перейдите по URL-адресу опубликованного приложения. URL-адрес приложения по умолчанию: `https://<web_app>.azurewebsites.net`.

# <a name="net-v12"></a>[\..NET (версии 12)](#tab/dotnet)

Выберите область **Отправка фотографий**, чтобы указать и отправить файл, или перетащите файлы в нее. При успешной отправке изображение исчезнет. Раздел **Созданные эскизы** будет оставаться пустым, пока мы не протестируем его позднее.

![Отправка фотографий в .NET](media/storage-upload-process-images/figure1.png)

В примере кода задача `UploadFileToStorage` в файле *Storagehelper.cs* используется для отправки изображений в контейнер *Изображения* в учетной записи хранения с помощью метода [UploadAsync](/dotnet/api/azure.storage.blobs.blobclient.uploadasync). В следующем примере кода содержится задача `UploadFileToStorage`.

```csharp
public static async Task<bool> UploadFileToStorage(Stream fileStream, string fileName,
                                                    AzureStorageConfig _storageConfig)
{
    // Create a URI to the blob
    Uri blobUri = new Uri("https://" +
                          _storageConfig.AccountName +
                          ".blob.core.windows.net/" +
                          _storageConfig.ImageContainer +
                          "/" + fileName);

    // Create StorageSharedKeyCredentials object by reading
    // the values from the configuration (appsettings.json)
    StorageSharedKeyCredential storageCredentials =
        new StorageSharedKeyCredential(_storageConfig.AccountName, _storageConfig.AccountKey);

    // Create the blob client.
    BlobClient blobClient = new BlobClient(blobUri, storageCredentials);

    // Upload the file
    await blobClient.UploadAsync(fileStream);

    return await Task.FromResult(true);
}
```

В предыдущей задаче используются следующие классы и методы:

| Class | Метод |
|-------|--------|
| [URI](/dotnet/api/system.uri) | [Uri constructor](/dotnet/api/system.uri.-ctor) |
| [StorageSharedKeyCredential](/dotnet/api/azure.storage.storagesharedkeycredential) | [StorageSharedKeyCredential(String, String) constructor](/dotnet/api/azure.storage.storagesharedkeycredential.-ctor) |
| [BlobClient](/dotnet/api/azure.storage.blobs.blobclient) | [UploadAsync](/dotnet/api/azure.storage.blobs.blobclient.uploadasync) |

# <a name="javascript-v12"></a>[JavaScript версии 12](#tab/javascript)

Чтобы выбрать файл, нажмите кнопку **Выбрать файл**, а затем **Загрузить изображение**. Раздел **Созданные эскизы** будет оставаться пустым, пока мы не протестируем его позднее.

![Отправка фотографий в Node.js](media/storage-upload-process-images/upload-app-nodejs.png)

В примере кода маршрут `post` отвечает за отправку изображения в контейнер BLOB-объектов. Маршрут использует модули для обработки отправки:

- [multer](https://github.com/expressjs/multer) реализует стратегию отправки для обработчика маршрутов.
- [into-stream](https://github.com/sindresorhus/into-stream) преобразует буфер в поток, как того требует [uploadStream](/javascript/api/%40azure/storage-blob/blockblobclient#uploadstream-readable--number--number--blockblobuploadstreamoptions-).

Когда файл отправляется в маршрут, содержимое файла остается в памяти до тех пор, пока файл не будет загружен в контейнер BLOB-объектов.

> [!IMPORTANT]
> Загрузка очень больших файлов в память может негативно повлиять на производительность вашего веб-приложения. Если вы ожидаете, что пользователи опубликуют большие файлы, можете просмотреть промежуточные файлы в файловой системе веб-сервера, а затем планировать загрузку в хранилище BLOB-объектов. После того как файлы попадают в хранилище BLOB-объектов, вы можете удалить их из файловой системы сервера.

```javascript
if (process.env.NODE_ENV !== 'production') {
  require('dotenv').config();
}

const {
  BlobServiceClient,
  StorageSharedKeyCredential,
  newPipeline
} = require('@azure/storage-blob');

const express = require('express');
const router = express.Router();
const containerName1 = 'thumbnails';
const multer = require('multer');
const inMemoryStorage = multer.memoryStorage();
const uploadStrategy = multer({ storage: inMemoryStorage }).single('image');
const getStream = require('into-stream');
const containerName2 = 'images';
const ONE_MEGABYTE = 1024 * 1024;
const uploadOptions = { bufferSize: 4 * ONE_MEGABYTE, maxBuffers: 20 };

const sharedKeyCredential = new StorageSharedKeyCredential(
  process.env.AZURE_STORAGE_ACCOUNT_NAME,
  process.env.AZURE_STORAGE_ACCOUNT_ACCESS_KEY);
const pipeline = newPipeline(sharedKeyCredential);

const blobServiceClient = new BlobServiceClient(
  `https://${process.env.AZURE_STORAGE_ACCOUNT_NAME}.blob.core.windows.net`,
  pipeline
);

const getBlobName = originalName => {
  // Use a random number to generate a unique file name, 
  // removing "0." from the start of the string.
  const identifier = Math.random().toString().replace(/0\./, '');
  return `${identifier}-${originalName}`;
};

router.get('/', async (req, res, next) => {

  let viewData;

  try {
    const containerClient = blobServiceClient.getContainerClient(containerName1);
    const listBlobsResponse = await containerClient.listBlobFlatSegment();

    for await (const blob of listBlobsResponse.segment.blobItems) {
      console.log(`Blob: ${blob.name}`);
    }

    viewData = {
      title: 'Home',
      viewName: 'index',
      accountName: process.env.AZURE_STORAGE_ACCOUNT_NAME,
      containerName: containerName1
    };

    if (listBlobsResponse.segment.blobItems.length) {
      viewData.thumbnails = listBlobsResponse.segment.blobItems;
    }
  } catch (err) {
    viewData = {
      title: 'Error',
      viewName: 'error',
      message: 'There was an error contacting the blob storage container.',
      error: err
    };
    res.status(500);
  } finally {
    res.render(viewData.viewName, viewData);
  }
});

router.post('/', uploadStrategy, async (req, res) => {
  const blobName = getBlobName(req.file.originalname);
  const stream = getStream(req.file.buffer);
  const containerClient = blobServiceClient.getContainerClient(containerName2);;
  const blockBlobClient = containerClient.getBlockBlobClient(blobName);

  try {
    await blockBlobClient.uploadStream(stream,
      uploadOptions.bufferSize, uploadOptions.maxBuffers,
      { blobHTTPHeaders: { blobContentType: "image/jpeg" } });
    res.render('success', { message: 'File uploaded to Azure Blob storage.' });
  } catch (err) {
    res.render('error', { message: err.message });
  }
});

module.exports = router;
```

---

## <a name="verify-the-image-is-shown-in-the-storage-account"></a>Проверка отображения изображения в учетной записи хранения

Войдите на [портал Azure](https://portal.azure.com). В меню слева выберите **Учетные записи хранения**, а затем имя своей учетной записи хранения. Выберите **Контейнеры**, а затем выберите контейнер **изображений**.

Убедитесь, что изображение отображается в контейнере.

![Список контейнеров изображений на портале Azure](media/storage-upload-process-images/figure13.png)

## <a name="test-thumbnail-viewing"></a>Тестирование просмотра эскизов

Для проверки просмотра эскизов отправьте изображение в контейнер **Эскизы**, чтобы убедиться, что приложение может считывать контейнер **Эскизы**.

Войдите на [портал Azure](https://portal.azure.com). В меню слева выберите **Учетные записи хранения**, а затем имя своей учетной записи хранения. Выберите **Контейнеры**, а затем выберите контейнер **эскизов**. Щелкните **Отправить**, чтобы открыть область **Передать BLOB-объект**.

С помощью средства выбора файлов выберите файл и щелкните **Отправить**.

Вернитесь к своему приложению и убедитесь, что изображение, отправленное в контейнер **эскизов**, отображается.

# <a name="net-v12"></a>[\..NET (версии 12)](#tab/dotnet)

![Приложение для изменения размера изображений .NET с новым изображением](media/storage-upload-process-images/figure2.png)

# <a name="javascript-v12"></a>[JavaScript версии 12](#tab/javascript)

![Приложение для изменения размера изображений Node.js с новым изображением](media/storage-upload-process-images/upload-app-nodejs-thumb.png)

---

Во второй части серии вы автоматизируете создание эскизных изображений, что позволит обойтись без этого изображения. В контейнере **эскизов** выберите отправленное изображение и щелкните **Удалить**, чтобы удалить его.

Включите сеть доставки содержимого, чтобы кэшировать содержимое учетной записи хранения. Дополнительные сведения см. в статье [Интеграция учетной записи хранения Azure с Azure CDN](../../cdn/cdn-create-a-storage-account-with-cdn.md).

## <a name="next-steps"></a>Дальнейшие действия

В первой части серии вы узнали, как настроить веб-приложение для взаимодействия с хранилищем.

Перейдите ко второй части руководства, чтобы узнать об использовании Сетки событий для активации функции Azure, изменяющей размер изображения.

> [!div class="nextstepaction"]
> [Automate resizing uploaded images using Event Grid](../../event-grid/resize-images-on-storage-blob-upload-event.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json) (Автоматическое изменение размера отправленных изображений с помощью службы "Сетка событий")
