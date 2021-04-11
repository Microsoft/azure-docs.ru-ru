---
title: Краткое руководство. Использование клиентской библиотеки Хранилища BLOB-объектов Azure версии 8 для Java
description: Создайте учетную запись хранения и контейнер в хранилище объектов (больших двоичных объектов). Затем используйте клиентскую библиотеку службы хранилища Azure для Java версии 8, чтобы отправить большой двоичный объект в службу хранилища Azure, скачать его и отобразить большие двоичные объекты в контейнере.
author: twooley
ms.custom: devx-track-java
ms.author: twooley
ms.date: 01/19/2021
ms.service: storage
ms.subservice: blobs
ms.topic: quickstart
ms.openlocfilehash: 49c55a2b565100e370ce561537c96a96b896f355
ms.sourcegitcommit: 02bc06155692213ef031f049f5dcf4c418e9f509
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/03/2021
ms.locfileid: "106280604"
---
# <a name="quickstart-manage-blobs-with-java-v8-sdk"></a>Краткое руководство. Управление большими двоичными объектами с помощью пакета SDK для Java версии 8

Из этого краткого руководства вы узнаете, как управлять большими двоичными объектами с использованием Java. Большие двоичные объекты — это объекты, которые могут содержать большие объемы текстовых или двоичных данных, включая изображения, документы, потоковое мультимедиа и архивные данные. Вы отправите, скачаете и отобразите большие двоичные объекты. Вы также создадите контейнеры, настроите для них разрешения и удалите их.

> [!NOTE]
> В этом кратком руководстве используется устаревшая версия клиентской библиотеки Хранилища BLOB-объектов Azure. Сведения о том, как начать работу с последней версией, см. в статье [Краткое руководство. Управление большими двоичными объектами с помощью пакета SDK для Java версии 12](storage-quickstart-blobs-java.md).

## <a name="prerequisites"></a>Предварительные требования

- Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio) бесплатно.
- Учетная запись хранения Azure. [Создание учетной записи хранения](../common/storage-account-create.md).
- Интегрированная среда разработки с интеграцией Maven. В этом руководстве используется среда разработки [Eclipse](https://www.eclipse.org/downloads/) с конфигурацией для разработчиков Java.

## <a name="download-the-sample-application"></a>Загрузка примера приложения

[Пример приложения](https://github.com/Azure-Samples/storage-blobs-java-quickstart) — это базовое консольное приложение.

Используйте команду [git](https://git-scm.com/), чтобы скачать копию приложения в среду разработки.

```bash
git clone https://github.com/Azure-Samples/storage-blobs-java-quickstart.git
```

Эта команда клонирует репозиторий в локальную папку git. Чтобы открыть проект, запустите Eclipse и закройте экран приветствия. Выберите **File** (Файл), затем щелкните **Open Projects from File System** (Открыть проекты из файловой системы). Убедитесь, что флажок **Detect and configure project natures** (Обнаружить и настроить свойства проекта) установлен. Выберите **Directory** (Каталог) и перейдите к папке хранения клонированного репозитория. В клонированном репозитории выберите папку **blobAzureApp**. Убедитесь, что проект **blobAzureApp** отображается как проект Eclipse, затем выберите **Finish** (Готово).

Когда проект будет импортирован, откройте файл **AzureApp.java** (расположенный в папке **blobQuickstart.blobAzureApp** в каталоге **main/src/java**) и замените `accountname` и `accountkey` в строке `storageConnectionString`. Затем запустите приложение. Конкретные инструкции по выполнению этих задач описаны в следующих разделах.

[!INCLUDE [storage-copy-connection-string-portal](../../../includes/storage-copy-connection-string-portal.md)]

## <a name="configure-your-storage-connection-string"></a>Настройка строки подключения хранилища

В приложении необходимо указать строку подключения для учетной записи хранения. Откройте файл **AzureApp.Java**. Найдите переменную `storageConnectionString` и вставьте значение строки подключения, скопированное в предыдущем разделе. Переменная `storageConnectionString` должна содержать данные, подобные приведенному ниже примеру кода:

```java
public static final String storageConnectionString =
"DefaultEndpointsProtocol=https;" +
"AccountName=<account-name>;" +
"AccountKey=<account-key>";
```

## <a name="run-the-sample"></a>Запуск примера

В этом примере приложения создается тестовый файл в каталоге по умолчанию (*C:\Users\<user>\AppData\Local\Temp* для пользователей Windows), затем он передается в Хранилище BLOB-объектов, после чего выводится список больших двоичных объектов в контейнере, а затем файл скачивается с новым именем, чтобы вы могли сравнить старый и новый файлы.

Запустите пример с помощью Maven в командной строке. Откройте оболочку и перейдите в папку **blobAzureApp** в клонированном каталоге. Затем введите `mvn compile exec:java`.

В следующем примере показаны выходные данные для запуска приложения в Windows.

```
Azure Blob storage quick start sample
Creating container: quickstartcontainer
Creating a sample file at: C:\Users\<user>\AppData\Local\Temp\sampleFile514658495642546986.txt
Uploading the sample file
URI of blob is: https://myexamplesacct.blob.core.windows.net/quickstartcontainer/sampleFile514658495642546986.txt
The program has completed successfully.
Press the 'Enter' key while in the console to delete the sample files, example container, and exit the application.

Deleting the container
Deleting the source, and downloaded files
```

Прежде чем продолжить, проверьте, есть ли пример файла в вашем каталоге по умолчанию (*C:\Users\<user>\AppData\Local\Temp* для пользователей Windows). Скопируйте URL-адрес BLOB-объекта из окна консоли и вставьте его в адресную строку браузера, чтобы просмотреть содержимое файла в хранилище BLOB-объектов. Если сравнить пример файла в каталоге с содержимым, хранящимся в хранилище BLOB-объектов, вы увидите, что они одинаковы.

  >[!NOTE]
  >Для просмотра файлов в хранилище BLOB-объектов можно также воспользоваться таким средством, как [обозреватель службы хранилища Azure](https://storageexplorer.com/?toc=%2fazure%2fstorage%2fblobs%2ftoc.json). Обозреватель службы хранилища Azure — это бесплатное кроссплатформенное средство для доступа к данным учетной записи хранения.

Проверив файлы, нажмите клавишу **ВВОД**, чтобы завершить работу с демонстрационной версией и удалить тестовые файлы. Теперь вы знаете, что делает этот пример. Давайте откроем файл **AzureApp.java** и изучим его код.

## <a name="understand-the-sample-code"></a>Разбор примера кода

Разберем пример кода, чтобы понять, как он работает.

### <a name="get-references-to-the-storage-objects"></a>Получение ссылок на объекты хранилища

Сначала необходимо создать ссылки на объекты, используемые для доступа к хранилищу BLOB-объектов и управлению им. Эти объекты зависят друг от друга — каждый объект используется следующим в списке объектом.

* Создайте экземпляр объекта [CloudStorageAccount](/java/api/com.microsoft.azure.management.storage.storageaccount), указывающий на учетную запись хранения.

    Объект **CloudStorageAccount** представляет вашу учетную запись хранения и позволяет настраивать и использовать ее свойства программным способом. С помощью **CloudStorageAccount** можно создать экземпляр объекта **CloudBlobClient**, который необходим для доступа к службе BLOB-объектов.

* Создайте экземпляр объекта **CloudBlobClient**, указывающий на [службу BLOB-объектов](/java/api/com.microsoft.azure.storage.blob.cloudblobclient) в учетной записи хранения.

    **CloudBlobClient** предоставляет точку доступа к службе BLOB-объектов и позволяет настраивать и использовать свойства хранилища BLOB-объектов программным способом. С помощью **CloudBlobClient** можно создать экземпляр объекта **CloudBlobContainer**, который необходим для создания контейнеров.

* Создайте экземпляр объекта [CloudBlobContainer](/java/api/com.microsoft.azure.storage.blob.cloudblobcontainer), представляющий контейнер, к которому осуществляется доступ. Используйте контейнеры для организации больших двоичных объектов аналогично папкам для упорядочения файлов на компьютере.

    Получив **CloudBlobContainer**, можно создать экземпляр объекта [CloudBlockBlob](/java/api/com.microsoft.azure.storage.blob.cloudblockblob), который указывает на нужный вам большой двоичный объект, и выполнить передачу, скачивание, копирование или другую операцию.

> [!IMPORTANT]
> Имена контейнеров должны состоять из знаков нижнего регистра. Дополнительные сведения о контейнерах см. в статье [Naming and Referencing Containers, Blobs, and Metadata](/rest/api/storageservices/naming-and-referencing-containers--blobs--and-metadata) (Именование контейнеров, больших двоичных объектов и метаданных и ссылка на них).

### <a name="create-a-container"></a>Создание контейнера

В этом разделе описано создание экземпляров объектов, создание контейнера и последующее задание разрешений для контейнера, что позволяет предоставить общий доступ к большим двоичным объектам по URL-адресу. Контейнер называется **quickstartcontainer**.

В этом примере используется [CreateIfNotExists](/java/api/com.microsoft.azure.storage.blob.cloudblobcontainer.createifnotexists), так как при каждом запуске примера требуется создавать новый контейнер. В рабочей среде, где в рамках приложения используется один и тот же контейнер, рекомендуется вызвать **CreateIfNotExists** только один раз. Вы можете также создать контейнер заранее, чтобы не создавать его в коде.

```java
// Parse the connection string and create a blob client to interact with Blob storage
storageAccount = CloudStorageAccount.parse(storageConnectionString);
blobClient = storageAccount.createCloudBlobClient();
container = blobClient.getContainerReference("quickstartcontainer");

// Create the container if it does not exist with public access.
System.out.println("Creating container: " + container.getName());
container.createIfNotExists(BlobContainerPublicAccessType.CONTAINER, new BlobRequestOptions(), new OperationContext());
```

### <a name="upload-blobs-to-the-container"></a>Отправка BLOB-объектов в контейнер

Чтобы отправить файл в блочный BLOB-объект, получите ссылку на этот объект в целевом контейнере. При наличии ссылки на BLOB-объект вы можете передать в него данные с помощью [CloudBlockBlob.Upload](/java/api/com.microsoft.azure.storage.blob.cloudblockblob.upload). Эта операция создает большой двоичный объект, если он еще не существует, или заменяет его, если существует.

В примере кода создается локальный файл, который будет использоваться для передачи и скачивания. Передаваемый файл сохраняется как **source**, и большому двоичному объекту присваивается имя в **blob**. В приведенном ниже примере файл отправляется в контейнер с именем **quickstart**.

```java
//Creating a sample file
sourceFile = File.createTempFile("sampleFile", ".txt");
System.out.println("Creating a sample file at: " + sourceFile.toString());
Writer output = new BufferedWriter(new FileWriter(sourceFile));
output.write("Hello Azure!");
output.close();

//Getting a blob reference
CloudBlockBlob blob = container.getBlockBlobReference(sourceFile.getName());

//Creating blob and uploading file to it
System.out.println("Uploading the sample file ");
blob.uploadFromFile(sourceFile.getAbsolutePath());
```

Есть несколько методов `upload`, которые можно использовать с хранилищем BLOB-объектов, включая [upload](/java/api/com.microsoft.azure.storage.blob.cloudblockblob.upload), [uploadBlock](/java/api/com.microsoft.azure.storage.blob.cloudblockblob.uploadblock), [uploadFullBlob](/java/api/com.microsoft.azure.storage.blob.cloudblockblob.uploadfullblob), [uploadStandardBlobTier](/java/api/com.microsoft.azure.storage.blob.cloudblockblob.uploadstandardblobtier) и [uploadText](/java/api/com.microsoft.azure.storage.blob.cloudblockblob.uploadtext). Для строки, например, лучше использовать метод `UploadText`, а не `Upload`.

Блочный BLOB-объект может представлять собой текстовый или двоичный файл любого типа. Страничные BLOB-объекты в основном используются для файлов VHD, применяемых для поддержки виртуальных машин IaaS. Используйте добавочные большие двоичные объекты для ведения журнала, например, если требуется выполнять запись в файл и добавлять дополнительные сведения. Большинство объектов, находящихся в хранилище BLOB-объектов, представляют собой блочные BLOB-объекты.

### <a name="list-the-blobs-in-a-container"></a>Перечисление BLOB-объектов в контейнере

Получить список файлов в контейнере можно с помощью метода [CloudBlobContainer.ListBlobs](/java/api/com.microsoft.azure.storage.blob.cloudblobcontainer.listblobs). Следующий код извлекает список BLOB-объектов, затем переходит по ним, отображая найденные URI. Можно скопировать URI из окна командной строки и вставить его в адресную строку браузера для просмотра файла.

```java
//Listing contents of container
for (ListBlobItem blobItem : container.listBlobs()) {
    System.out.println("URI of blob is: " + blobItem.getUri());
}
```

### <a name="download-blobs"></a>Скачивание больших двоичных объектов

Скачайте большие двоичные объекты на локальный диск с помощью метода [CloudBlob.DownloadToFile](/java/api/com.microsoft.azure.storage.blob.cloudblob.downloadtofile).

Следующий код скачивает BLOB-объект, отправленный в предыдущем разделе, добавляя к имени BLOB-объекта суффикс "_DOWNLOADED", чтобы вы увидели оба файла на локальном диске.

```java
// Download blob. In most cases, you would have to retrieve the reference
// to cloudBlockBlob here. However, we created that reference earlier, and
// haven't changed the blob we're interested in, so we can reuse it.
// Here we are creating a new file to download to. Alternatively you can also pass in the path as a string into downloadToFile method: blob.downloadToFile("/path/to/new/file").
downloadedFile = new File(sourceFile.getParentFile(), "downloadedFile.txt");
blob.downloadToFile(downloadedFile.getAbsolutePath());
```

### <a name="clean-up-resources"></a>Очистка ресурсов

Если вам больше не нужны отправленные большие двоичные объекты, удалите весь контейнер с помощью метода [CloudBlobContainer.DeleteIfExists](/java/api/com.microsoft.azure.storage.blob.cloudblobcontainer.deleteifexists). Он также удаляет файлы в контейнере.

```java
try {
if(container != null)
    container.deleteIfExists();
} catch (StorageException ex) {
System.out.println(String.format("Service error. Http code: %d and error code: %s", ex.getHttpStatusCode(), ex.getErrorCode()));
}

System.out.println("Deleting the source, and downloaded files");

if(downloadedFile != null)
downloadedFile.deleteOnExit();

if(sourceFile != null)
sourceFile.deleteOnExit();
```

## <a name="next-steps"></a>Дальнейшие действия

В этом кратком руководстве описано, как передавать файлы между локальным диском и хранилищем BLOB-объектов Azure с помощью Java. Чтобы узнать подробнее о работе с Java, перейдите в репозиторий исходного кода на GitHub.

> [!div class="nextstepaction"]
> [Справочник по API Java](/java/api/overview/azure/storage?view=azure-java-legacy&preserve-view=true)
> [Примеры кода для Java](../common/storage-samples-java.md)