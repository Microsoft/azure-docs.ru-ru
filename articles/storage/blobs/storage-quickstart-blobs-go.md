---
title: Краткое руководство Azure. Создание большого двоичного объекта в хранилище объектов с помощью Go | Документация Майкрософт
description: В рамках этого краткого руководства вы создадите учетную запись хранения и контейнер в хранилище объектов (больших двоичных объектов). Затем используете клиентскую библиотеку службы хранилища для языка Go, чтобы отправить большой двоичный объект в службу хранилища Azure, скачать его и составить список больших двоичных объектов в контейнере.
author: twooley
ms.author: twooley
ms.date: 11/14/2018
ms.service: storage
ms.subservice: blobs
ms.topic: quickstart
ms.openlocfilehash: aa1c4aea2966e70728fbde22fdcbe1e8d78cb166
ms.sourcegitcommit: 02bc06155692213ef031f049f5dcf4c418e9f509
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/03/2021
ms.locfileid: "106280621"
---
# <a name="quickstart-upload-download-and-list-blobs-using-go"></a>Краткое руководство по передаче, скачиванию и составлению списка больших двоичных объектов с помощью языка Go

Из этого краткого руководства вы узнаете, как передать, скачать и создать список блочных BLOB-объектов в контейнере в хранилище BLOB-объектов Azure с помощью языка Go. 

## <a name="prerequisites"></a>предварительные требования

[!INCLUDE [storage-quickstart-prereq-include](../../../includes/storage-quickstart-prereq-include.md)]

Прежде чем приступить к работе, убедитесь, что у вас установлены следующие дополнительные компоненты:
 
* [Go версии 1.8 или выше](https://golang.org/dl/).
* [Пакет SDK Azure Storage Blob для Go](https://github.com/azure/azure-storage-blob-go/), установленный с помощью следующей команды:

    ```
    go get -u github.com/Azure/azure-storage-blob-go/azblob
    ``` 

    > [!NOTE]
    > Убедитесь, что `Azure` указывается в URL-адресе с прописной буквы, иначе при работе с пакетом SDK могут возникнуть проблемы с импортом. `Azure` также нужно указывать с прописной буквы в операторах импорта.
    
## <a name="download-the-sample-application"></a>Загрузка примера приложения
В этом кратком руководстве как [пример](https://github.com/Azure-Samples/storage-blobs-go-quickstart.git) используется простое приложение Go.  

Используйте команду [git](https://git-scm.com/), чтобы скачать копию приложения в среду разработки. 

```bash
git clone https://github.com/Azure-Samples/storage-blobs-go-quickstart 
```

Эта команда клонирует репозиторий в локальную папку git. Чтобы открыть пример Go для хранилища BLOB-объектов, найдите файл storage-quickstart.go.  

[!INCLUDE [storage-copy-account-key-portal](../../../includes/storage-copy-account-key-portal.md)]

## <a name="configure-your-storage-connection-string"></a>Настройка строки подключения хранилища
Этому решению требуется, чтобы имя и ключ учетной записи хранения безопасно хранились в переменных среды локального компьютера, на котором выполняется пример. Чтобы создать переменную среды, выполните один из приведенных ниже примеров в зависимости от операционной системы.

# <a name="linux"></a>[Linux](#tab/linux)

```
export AZURE_STORAGE_ACCOUNT="<youraccountname>"
export AZURE_STORAGE_ACCESS_KEY="<youraccountkey>"
```

# <a name="windows"></a>[Windows](#tab/windows)

```
setx AZURE_STORAGE_ACCOUNT "<youraccountname>"
setx AZURE_STORAGE_ACCESS_KEY "<youraccountkey>"
```

---

## <a name="run-the-sample"></a>Запуск примера
Этот пример создает тестовый файл в текущей папке, отправляет тестовый файл в хранилище BLOB-объектов, создает список больших двоичных объектов в контейнере и загружает файл в буфер. 

Чтобы выполнить пример, используйте следующую команду: 

```go run storage-quickstart.go```

Ниже представлен пример выходных данных, которые возвращаются при выполнении этого приложения.
  
```
Azure Blob storage quick start sample
Creating a container named quickstart-5568059279520899415
Creating a dummy file to test the upload and download
Uploading the file with blob name: 630910657703031215
Blob name: 630910657703031215
Downloaded the blob: hello world
this is a blob
Press the enter key to delete the sample files, example container, and exit the application.
```
Теперь нажмите клавишу, чтобы пример программы удалил контейнер хранилища и файлы. 

> [!TIP]
> Для просмотра файлов в хранилище BLOB-объектов можно также воспользоваться таким средством, как [обозреватель службы хранилища Azure](https://storageexplorer.com). Обозреватель службы хранилища Azure — это бесплатное кроссплатформенное средство для доступа к данным учетной записи хранения. 
>

## <a name="understand-the-sample-code"></a>Разбор примера кода

Разберем пример кода, чтобы понять, как он работает.

### <a name="create-containerurl-and-bloburl-objects"></a>Создание объектов ContainerURL и BlobURL
Сначала необходимо создать ссылки на объекты ContainerURL и BlobURL, необходимые при доступе к хранилищу BLOB-объектов и управлении им. Эти объекты предоставляют низкоуровневые API, такие как Create, Upload и Download, для выдачи REST API.

* Используйте структуру [**SharedKeyCredential**](https://godoc.org/github.com/Azure/azure-storage-blob-go/azblob#SharedKeyCredential) для хранения учетных данных. 

* Создайте [**конвейер**](https://godoc.org/github.com/Azure/azure-storage-blob-go/azblob#NewPipeline) с помощью учетных данных и параметров. Конвейер задает такие элементы, как политики повтора, ведение журнала, десериализация полезных данных HTTP-ответа и многое другое.  

* Создайте объект [**ContainerURL**](https://godoc.org/github.com/Azure/azure-storage-blob-go/azblob#ContainerURL) и [**BlobURL**](https://godoc.org/github.com/Azure/azure-storage-blob-go/azblob#BlobURL), чтобы выполнить операции в контейнере (Create) и больших двоичных объектах (Upload и Download).


Получив объект ContainerURL, вы можете создать объект **BlobURL**, который указывает на большой двоичный объект и выполняет такие операции, как отправка, скачивание и копирование.

> [!IMPORTANT]
> Имена контейнеров должны состоять из знаков нижнего регистра. Дополнительные сведения об именовании контейнеров и больших двоичных объектов см. в статье [Naming and Referencing Containers, Blobs, and Metadata](/rest/api/storageservices/naming-and-referencing-containers--blobs--and-metadata) (Именование контейнеров, больших двоичных объектов и метаданных и ссылка на них).

В этом разделе показано, как создать контейнер. Контейнер называется **quickstartblobs-[произвольная_строка]** . 

```go 
// From the Azure portal, get your storage account name and key and set environment variables.
accountName, accountKey := os.Getenv("AZURE_STORAGE_ACCOUNT"), os.Getenv("AZURE_STORAGE_ACCESS_KEY")
if len(accountName) == 0 || len(accountKey) == 0 {
    log.Fatal("Either the AZURE_STORAGE_ACCOUNT or AZURE_STORAGE_ACCESS_KEY environment variable is not set")
}

// Create a default request pipeline using your storage account name and account key.
credential, err := azblob.NewSharedKeyCredential(accountName, accountKey)
if err != nil {
    log.Fatal("Invalid credentials with error: " + err.Error())
}
p := azblob.NewPipeline(credential, azblob.PipelineOptions{})

// Create a random string for the quick start container
containerName := fmt.Sprintf("quickstart-%s", randomString())

// From the Azure portal, get your storage account blob service URL endpoint.
URL, _ := url.Parse(
    fmt.Sprintf("https://%s.blob.core.windows.net/%s", accountName, containerName))

// Create a ContainerURL object that wraps the container URL and a request
// pipeline to make requests.
containerURL := azblob.NewContainerURL(*URL, p)

// Create the container
fmt.Printf("Creating a container named %s\n", containerName)
ctx := context.Background() // This example uses a never-expiring context
_, err = containerURL.Create(ctx, azblob.Metadata{}, azblob.PublicAccessNone)
handleErrors(err)
```
### <a name="upload-blobs-to-the-container"></a>Отправка BLOB-объектов в контейнер

Хранилище BLOB-объектов поддерживает блочные, добавочные и страничные BLOB-объекты. Чаще всего используются блочные BLOB-объекты. Именно этот тип представлен в этом кратком руководстве.  

Чтобы отправить файл в большой двоичный объект, откройте файл с помощью **os.Open**. Затем вы можете отправить файл по указанному пути, используя один из REST API: Upload (PutBlob), StageBlock/CommitBlockList (PutBlock/PutBlockList). 

Кроме того, пакет SDK предоставляет [API высокого уровня](https://github.com/Azure/azure-storage-blob-go/blob/master/azblob/highlevel.go), которые созданы на основе низкоуровневых интерфейсов REST API. Например, с помощью операций StageBlock (PutBlock) функция ***UploadFileToBlockBlob*** одновременно передает файлы блоками, оптимизируя тем самым пропускную способность. Если размер файла менее 256 МБ, передача осуществляется одной транзакцией с помощью операции Upload (PutBlob).

В приведенном ниже примере файл отправляется в контейнер с именем **quickstartblobs-[произвольная_строка]** .

```go
// Create a file to test the upload and download.
fmt.Printf("Creating a dummy file to test the upload and download\n")
data := []byte("hello world this is a blob\n")
fileName := randomString()
err = ioutil.WriteFile(fileName, data, 0700)
handleErrors(err)

// Here's how to upload a blob.
blobURL := containerURL.NewBlockBlobURL(fileName)
file, err := os.Open(fileName)
handleErrors(err)

// You can use the low-level Upload (PutBlob) API to upload files. Low-level APIs are simple wrappers for the Azure Storage REST APIs.
// Note that Upload can upload up to 256MB data in one shot. Details: https://docs.microsoft.com/rest/api/storageservices/put-blob
// To upload more than 256MB, use StageBlock (PutBlock) and CommitBlockList (PutBlockList) functions. 
// Following is commented out intentionally because we will instead use UploadFileToBlockBlob API to upload the blob
// _, err = blobURL.Upload(ctx, file, azblob.BlobHTTPHeaders{ContentType: "text/plain"}, azblob.Metadata{}, azblob.BlobAccessConditions{})
// handleErrors(err)

// The high-level API UploadFileToBlockBlob function uploads blocks in parallel for optimal performance, and can handle large files as well.
// This function calls StageBlock/CommitBlockList for files larger 256 MBs, and calls Upload for any file smaller
fmt.Printf("Uploading the file with blob name: %s\n", fileName)
_, err = azblob.UploadFileToBlockBlob(ctx, file, blobURL, azblob.UploadToBlockBlobOptions{
    BlockSize:   4 * 1024 * 1024,
    Parallelism: 16})
handleErrors(err)
```

### <a name="list-the-blobs-in-a-container"></a>Перечисление BLOB-объектов в контейнере

Список файлов в контейнере можно получить с помощью метода **ListBlobs** в **ContainerURL**. ListBlobs возвращает один сегмент больших двоичных объектов (до 5000), начиная с указанного **маркера**. Используйте пустой маркер, чтобы запустить перечисление с самого начала. Имена больших двоичных объектов возвращаются в лексикографическом порядке. После получения обработайте сегмент, а затем вызовите метод ListBlobs еще раз, передав ранее возвращенный маркер.  

```go
// List the container that we have created above
fmt.Println("Listing the blobs in the container:")
for marker := (azblob.Marker{}); marker.NotDone(); {
    // Get a result segment starting with the blob indicated by the current Marker.
    listBlob, err := containerURL.ListBlobsFlatSegment(ctx, marker, azblob.ListBlobsSegmentOptions{})
    handleErrors(err)

    // ListBlobs returns the start of the next segment; you MUST use this to get
    // the next segment (after processing the current result segment).
    marker = listBlob.NextMarker

    // Process the blobs returned in this result segment (if the segment is empty, the loop body won't execute)
    for _, blobInfo := range listBlob.Segment.BlobItems {
        fmt.Print(" Blob name: " + blobInfo.Name + "\n")
    }
}
```

### <a name="download-the-blob"></a>Скачивание большого двоичного объекта

Загрузите большие двоичные объекты с помощью функции низкого уровня **Download** в BlobURL. Эта команда возвращает структуру **DownloadResponse**. Запустите в структуре функцию **Body**, чтобы получить поток для чтения данных **RetryReader**. Если при чтении подключение завершится ошибкой, будут созданы дополнительные запросы для повторной установки подключения и продолжения чтения. Если указать RetryReaderOption с параметром MaxRetryRequests, для которого установлено значение 0 (по умолчанию), будет возвращен исходный текст ответа и повторные попытки не выполняются. Кроме того, для упрощения кода используйте API-интерфейсы высокого уровня **DownloadBlobToBuffer** или **DownloadBlobToFile**.

Следующий код скачивает большой двоичный объект с помощью функции **Download**. Содержимое большого двоичного объекта записывается в буфер и отображается в консоли.

```go
// Here's how to download the blob
downloadResponse, err := blobURL.Download(ctx, 0, azblob.CountToEnd, azblob.BlobAccessConditions{}, false)

// NOTE: automatically retries are performed if the connection fails
bodyStream := downloadResponse.Body(azblob.RetryReaderOptions{MaxRetryRequests: 20})

// read the body into a buffer
downloadedData := bytes.Buffer{}
_, err = downloadedData.ReadFrom(bodyStream)
handleErrors(err)
```

### <a name="clean-up-resources"></a>Очистка ресурсов
Если большие двоичные объекты, отправленные при работе с этим руководством, больше не нужны, удалите весь контейнер с помощью метода **Delete**. 

```go
// Cleaning up the quick start by deleting the container and the file created locally
fmt.Printf("Press enter key to delete the sample files, example container, and exit the application.\n")
bufio.NewReader(os.Stdin).ReadBytes('\n')
fmt.Printf("Cleaning up.\n")
containerURL.Delete(ctx, azblob.ContainerAccessConditions{})
file.Close()
os.Remove(fileName)
```

## <a name="resources-for-developing-go-applications-with-blobs"></a>Ресурсы для разработки приложений Go с большими двоичными объектами

Ознакомьтесь со следующими дополнительными ресурсами для разработки Go с использованием хранилища BLOB-объектов:

- Просмотрите и установите [исходный код клиентской библиотеки Go](https://github.com/Azure/azure-storage-blob-go) для службы хранилища Azure на сайте GitHub.
- Изучите [примеры для хранилища BLOB-объектов](https://godoc.org/github.com/Azure/azure-storage-blob-go/azblob#pkg-examples), написанные с использованием клиентской библиотеки Go.

## <a name="next-steps"></a>Дальнейшие действия
 
В этом кратком руководстве вы узнали, как передавать файлы между локальным диском и хранилищем BLOB-объектов Azure с помощью Go. Дополнительные сведения о пакете SDK для Azure Storage Blob см. в [этом репозитории с исходным кодом](https://github.com/Azure/azure-storage-blob-go/) и[справочнике по API](https://godoc.org/github.com/Azure/azure-storage-blob-go/azblob).