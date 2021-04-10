---
title: Краткое руководство. Использование библиотеки Хранилища BLOB-объектов Azure версии 12 для Java
description: В этом кратком руководстве описывается, как использовать клиентскую библиотеку Хранилища BLOB-объектов Azure версии 12 для Java для создания контейнера и большого двоичного объекта в хранилище BLOB-объектов. Далее вы узнаете, как скачать большой двоичный объект на локальный компьютер и как получить список всех больших двоичных объектов в контейнере.
author: mhopkins-msft
ms.custom: devx-track-java
ms.author: mhopkins
ms.date: 12/01/2020
ms.service: storage
ms.subservice: blobs
ms.topic: quickstart
ms.openlocfilehash: b5c34cea5d8222a246462bfadde66fd8a5ddbec7
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "99054709"
---
# <a name="quickstart-manage-blobs-with-java-v12-sdk"></a>Краткое руководство. Управление большими двоичными объектами с помощью пакета SDK для Java версии 12

Из этого краткого руководства вы узнаете, как управлять большими двоичными объектами с использованием Java. Большие двоичные объекты — это объекты, которые могут содержать большие объемы текстовых или двоичных данных, включая изображения, документы, потоковое мультимедиа и архивные данные. Вы научитесь отправлять и скачивать большие двоичные объекты, получать список таких объектов, а также создавать и удалять контейнеры.

Дополнительные ресурсы:

* [Справочная документация по API](/java/api/overview/azure/storage-blob-readme)
* [Исходный код библиотеки](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/storage/azure-storage-blob)
* [Пакет (Maven)](https://mvnrepository.com/artifact/com.azure/azure-storage-blob)
* [Примеры](../common/storage-samples-java.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json#blob-samples)

## <a name="prerequisites"></a>Предварительные требования

- Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio) бесплатно.
- Учетная запись хранения Azure. [Создание учетной записи хранения](../common/storage-account-create.md).
- [комплект SDK для Java (JDK)](/java/azure/jdk/) версии 8 или более поздней версии.
- [Apache Maven](https://maven.apache.org/download.cgi).

[!INCLUDE [storage-multi-protocol-access-preview](../../../includes/storage-multi-protocol-access-preview.md)]

## <a name="setting-up"></a>Настройка

В этом разделе рассматривается подготовка проекта для работы с клиентской библиотекой Хранилища BLOB-объектов Azure версии 12 для Java.

### <a name="create-the-project"></a>Создание проекта

Создайте приложение Java с именем *blob-quickstart-v12*.

1. В окне консоли (командная строка, PowerShell или Bash) с помощью Maven создайте консольное приложение с именем *blob-quickstart-v12*. Введите следующую команду **mvn**, чтобы создать проект Java "Hello World".

    # <a name="powershell"></a>[PowerShell](#tab/powershell)

    ```powershell
    mvn archetype:generate `
        --define interactiveMode=n `
        --define groupId=com.blobs.quickstart `
        --define artifactId=blob-quickstart-v12 `
        --define archetypeArtifactId=maven-archetype-quickstart `
        --define archetypeVersion=1.4
    ```

    # <a name="bash"></a>[Bash](#tab/bash)

    ```bash
    mvn archetype:generate \
        --define interactiveMode=n \
        --define groupId=com.blobs.quickstart \
        --define artifactId=blob-quickstart-v12 \
        --define archetypeArtifactId=maven-archetype-quickstart \
        --define archetypeVersion=1.4
    ```

    ---

1. Примерный результат создания проекта показан ниже.

    ```console
    [INFO] Scanning for projects...
    [INFO]
    [INFO] ------------------< org.apache.maven:standalone-pom >-------------------
    [INFO] Building Maven Stub Project (No POM) 1
    [INFO] --------------------------------[ pom ]---------------------------------
    [INFO]
    [INFO] >>> maven-archetype-plugin:3.1.2:generate (default-cli) > generate-sources @ standalone-pom >>>
    [INFO]
    [INFO] <<< maven-archetype-plugin:3.1.2:generate (default-cli) < generate-sources @ standalone-pom <<<
    [INFO]
    [INFO]
    [INFO] --- maven-archetype-plugin:3.1.2:generate (default-cli) @ standalone-pom ---
    [INFO] Generating project in Batch mode
    [INFO] ----------------------------------------------------------------------------
    [INFO] Using following parameters for creating project from Archetype: maven-archetype-quickstart:1.4
    [INFO] ----------------------------------------------------------------------------
    [INFO] Parameter: groupId, Value: com.blobs.quickstart
    [INFO] Parameter: artifactId, Value: blob-quickstart-v12
    [INFO] Parameter: version, Value: 1.0-SNAPSHOT
    [INFO] Parameter: package, Value: com.blobs.quickstart
    [INFO] Parameter: packageInPathFormat, Value: com/blobs/quickstart
    [INFO] Parameter: version, Value: 1.0-SNAPSHOT
    [INFO] Parameter: package, Value: com.blobs.quickstart
    [INFO] Parameter: groupId, Value: com.blobs.quickstart
    [INFO] Parameter: artifactId, Value: blob-quickstart-v12
    [INFO] Project created from Archetype in dir: C:\QuickStarts\blob-quickstart-v12
    [INFO] ------------------------------------------------------------------------
    [INFO] BUILD SUCCESS
    [INFO] ------------------------------------------------------------------------
    [INFO] Total time:  7.056 s
    [INFO] Finished at: 2019-10-23T11:09:21-07:00
    [INFO] ------------------------------------------------------------------------
        ```

1. Switch to the newly created *blob-quickstart-v12* folder.

   ```console
   cd blob-quickstart-v12
   ```

1. В каталоге *blob-quickstart-v12* создайте каталог *data*. Это каталог для создания и хранения файлов данных больших двоичных объектов.

    ```console
    mkdir data
    ```

### <a name="install-the-package"></a>Установка пакета

Откройте файл *pom.xml* в текстовом редакторе. Добавьте приведенный ниже элемент зависимости в группу зависимостей.

```xml
<dependency>
    <groupId>com.azure</groupId>
    <artifactId>azure-storage-blob</artifactId>
    <version>12.6.0</version>
</dependency>
```

### <a name="set-up-the-app-framework"></a>Настройка платформы приложения

Из каталога проекта:

1. Перейдите в каталог */src/main/java/com/blobs/quickstart*.
1. Откройте файл *App.java* в редакторе.
1. Удалите оператор `System.out.println("Hello world!");`.
1. Добавьте директивы `import`.

Вот этот код:

```java
package com.blobs.quickstart;

/**
 * Azure blob storage v12 SDK quickstart
 */
import com.azure.storage.blob.*;
import com.azure.storage.blob.models.*;
import java.io.*;

public class App
{
    public static void main( String[] args ) throws IOException
    {
    }
}
```

[!INCLUDE [storage-quickstart-credentials-include](../../../includes/storage-quickstart-credentials-include.md)]

## <a name="object-model"></a>Объектная модель

Хранилище BLOB-объектов Azure оптимизировано для хранения больших объемов неструктурированных данных. Неструктурированные данные — это данные, которые не соответствуют определенной модели данных или определению, например текстовых или двоичных данных. В хранилище BLOB-объектов предлагается три типа ресурсов:

* учетная запись хранения;
* контейнер в учетной записи хранения;
* большой двоичный объект в контейнере.

На следующей схеме показана связь между этими ресурсами.

![Схема архитектуры службы хранилища BLOB-объектов](./media/storage-blobs-introduction/blob1.png)

Используйте следующие классы Java для взаимодействия с этими ресурсами.

* [BlobServiceClient](/java/api/com.azure.storage.blob.blobserviceclient). Класс `BlobServiceClient` позволяет управлять ресурсами службы хранилища Azure и контейнерами больших двоичных объектов. Учетная запись хранения предоставляет пространство имен верхнего уровня для службы BLOB-объектов.
* Класс [ предоставляет API гибкого конструктора для упрощения настройки и создания экземпляров объектов ](/java/api/com.azure.storage.blob.blobserviceclientbuilder).
* [BlobContainerClient](/java/api/com.azure.storage.blob.blobcontainerclient). Класс `BlobContainerClient` позволяет управлять контейнерами службы хранилища Azure и содержащимися в них большими двоичными объектами.
* Класс [ позволяет управлять большими двоичными объектами службы хранилища Azure.
* [BlobItem](/java/api/com.azure.storage.blob.models.blobitem). Класс `BlobItem` представляет отдельные большие двоичные объекты, возвращаемые при вызове метода [listBlobs](/java/api/com.azure.storage.blob.blobcontainerclient.listblobs).

## <a name="code-examples"></a>Примеры кода

В этих примерах фрагментов кода показано, как выполнять следующие действия с помощью клиентской библиотеки Хранилища BLOB-объектов Azure для Java:

* [Получение строки подключения](#get-the-connection-string)
* [Создание контейнера](#create-a-container)
* [отправка больших двоичных объектов в контейнер](#upload-blobs-to-a-container);
* [перечисление больших двоичных объектов в контейнере](#list-the-blobs-in-a-container);
* [скачивание больших двоичных объектов](#download-blobs);
* [Удаление контейнера](#delete-a-container)

### <a name="get-the-connection-string"></a>Получение строки подключения

Приведенный ниже код извлекает строку подключения для учетной записи хранения из переменной среды, созданной в разделе [Настройка строки подключения хранилища](#configure-your-storage-connection-string).

Добавьте этот код в метод `Main`.

```java
System.out.println("Azure Blob Storage v12 - Java quickstart sample\n");

// Retrieve the connection string for use with the application. The storage
// connection string is stored in an environment variable on the machine
// running the application called AZURE_STORAGE_CONNECTION_STRING. If the environment variable
// is created after the application is launched in a console or with
// Visual Studio, the shell or application needs to be closed and reloaded
// to take the environment variable into account.
String connectStr = System.getenv("AZURE_STORAGE_CONNECTION_STRING");
```

### <a name="create-a-container"></a>Создание контейнера

Выберите имя нового контейнера. Приведенный ниже код добавляет к имени контейнера значение UUID, чтобы сделать это имя уникальным.

> [!IMPORTANT]
> Имена контейнеров должны состоять из знаков нижнего регистра. Дополнительные сведения об именовании контейнеров и больших двоичных объектов см. в статье [Naming and Referencing Containers, Blobs, and Metadata](/rest/api/storageservices/naming-and-referencing-containers--blobs--and-metadata) (Именование контейнеров, больших двоичных объектов и метаданных и ссылка на них).

Затем создайте экземпляр класса [BlobContainerClient](/java/api/com.azure.storage.blob.blobcontainerclient) и вызовите метод [create](/java/api/com.azure.storage.blob.blobcontainerclient.create), чтобы создать контейнер в учетной записи хранения.

Добавьте следующий код в конец метода `Main`.

```java
// Create a BlobServiceClient object which will be used to create a container client
BlobServiceClient blobServiceClient = new BlobServiceClientBuilder().connectionString(connectStr).buildClient();

//Create a unique name for the container
String containerName = "quickstartblobs" + java.util.UUID.randomUUID();

// Create the container and return a container client object
BlobContainerClient containerClient = blobServiceClient.createBlobContainer(containerName);
```

### <a name="upload-blobs-to-a-container"></a>Отправка больших двоичных объектов в контейнер

Приведенный ниже фрагмент кода:

1. Создает текстовый файл в локальном каталоге *data*.
1. Возвращает ссылку на объект [BlobClient](/java/api/com.azure.storage.blob.blobclient), вызывая метод [getBlobClient](/java/api/com.azure.storage.blob.blobcontainerclient.getblobclient) для контейнера из раздела [Создание контейнера](#create-a-container).
1. Передает локальный текстовый файл в большой двоичный объект, вызывая метод [uploadFromFile](/java/api/com.azure.storage.blob.blobclient.uploadfromfile). С помощью этого метода создается большой двоичный объект, если он не был создан ранее. Если он имеется, замещение не происходит.

Добавьте следующий код в конец метода `Main`.

```java
// Create a local file in the ./data/ directory for uploading and downloading
String localPath = "./data/";
String fileName = "quickstart" + java.util.UUID.randomUUID() + ".txt";
File localFile = new File(localPath + fileName);

// Write text to the file
FileWriter writer = new FileWriter(localPath + fileName, true);
writer.write("Hello, World!");
writer.close();

// Get a reference to a blob
BlobClient blobClient = containerClient.getBlobClient(fileName);

System.out.println("\nUploading to Blob storage as blob:\n\t" + blobClient.getBlobUrl());

// Upload the blob
blobClient.uploadFromFile(localPath + fileName);
```

### <a name="list-the-blobs-in-a-container"></a>Перечисление BLOB-объектов в контейнере

Выведите список больших двоичных объектов в контейнере, вызвав метод [listBlobs](/java/api/com.azure.storage.blob.blobcontainerclient.listblobs). В этом случае в контейнер был добавлен лишь один большой двоичный объект, поэтому операция перечисления возвращает только его.

Добавьте следующий код в конец метода `Main`.

```java
System.out.println("\nListing blobs...");

// List the blob(s) in the container.
for (BlobItem blobItem : containerClient.listBlobs()) {
    System.out.println("\t" + blobItem.getName());
}
```

### <a name="download-blobs"></a>Скачивание больших двоичных объектов

Скачайте созданный ранее большой двоичный объект, вызвав метод [downloadToFile](/java/api/com.azure.storage.blob.specialized.blobclientbase.downloadtofile). Пример кода добавляет суффикс "DOWNLOAD" к имени файла, чтобы в локальной файловой системе можно было просмотреть оба файла.

Добавьте следующий код в конец метода `Main`.

```java
// Download the blob to a local file
// Append the string "DOWNLOAD" before the .txt extension so that you can see both files.
String downloadFileName = fileName.replace(".txt", "DOWNLOAD.txt");
File downloadedFile = new File(localPath + downloadFileName);

System.out.println("\nDownloading blob to\n\t " + localPath + downloadFileName);

blobClient.downloadToFile(localPath + downloadFileName);
```

### <a name="delete-a-container"></a>Удаление контейнера

Следующий код очищает созданные приложением ресурсы, полностью удаляя контейнер с помощью метода [delete](/java/api/com.azure.storage.blob.blobcontainerclient.delete). Он также удаляет локальные файлы, созданные приложением.

Приложение приостанавливается для ввода пользователя, вызывая `System.console().readLine()`, перед удалением большого двоичного объекта, контейнера и локальных файлов. Это хорошая возможность проверить правильность создания ресурсов перед их удалением.

Добавьте следующий код в конец метода `Main`.

```java
// Clean up
System.out.println("\nPress the Enter key to begin clean up");
System.console().readLine();

System.out.println("Deleting blob container...");
containerClient.delete();

System.out.println("Deleting the local source and downloaded files...");
localFile.delete();
downloadedFile.delete();

System.out.println("Done");
```

## <a name="run-the-code"></a>Выполнение кода

В этом приложении тестовый файл создается в локальной папке, а затем передается в хранилище BLOB-объектов. После этого выводится список больших двоичных объектов в контейнере, а затем файл загружается с новым именем, чтобы можно было сравнить старый и новый файлы.

Перейдите в каталог, содержащий файл *pom.xml*, и скомпилируйте проект с помощью следующей команды `mvn`.

```console
mvn compile
```

Затем выполните сборку пакета.

```console
mvn package
```

Выполните следующую команду `mvn` для запуска приложения.

```console
mvn exec:java -Dexec.mainClass="com.blobs.quickstart.App" -Dexec.cleanupDaemonThreads=false
```

Вы должны увидеть выходные данные приложения, как показано ниже.

```output
Azure Blob Storage v12 - Java quickstart sample

Uploading to Blob storage as blob:
        https://mystorageacct.blob.core.windows.net/quickstartblobsf9aa68a5-260e-47e6-bea2-2dcfcfa1fd9a/quickstarta9c3a53e-ae9d-4863-8b34-f3d807992d65.txt

Listing blobs...
        quickstarta9c3a53e-ae9d-4863-8b34-f3d807992d65.txt

Downloading blob to
        ./data/quickstarta9c3a53e-ae9d-4863-8b34-f3d807992d65DOWNLOAD.txt

Press the Enter key to begin clean up

Deleting blob container...
Deleting the local source and downloaded files...
Done
```

Прежде чем начать удаление, проверьте наличие двух файлов в папке *data*. Вы можете открыть их и убедиться, что они идентичны.

После проверки файлов нажмите клавишу **ВВОД**, чтобы завершить работу с демонстрационной версией и удалить тестовые файлы.

## <a name="next-steps"></a>Дальнейшие действия

В этом кратком руководстве вы узнали, как передавать и скачивать большие двоичные объекты, а также выводить их список с помощью Java.

Чтобы просмотреть примеры приложений для хранилища BLOB-объектов, перейдите к следующему разделу:

> [!div class="nextstepaction"]
> [Примеры для пакета SDK Хранилища BLOB-объектов Azure версии 12 для Java](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/storage/azure-storage-blob/src/samples/java/com/azure/storage/blob)

* Чтобы узнать больше, ознакомьтесь с [пакетом SDK Azure для Java](https://github.com/Azure/azure-sdk-for-java/blob/master/README.md).
* Учебники, примеры, краткие руководства и другую документацию можно найти на странице [Azure для разработчиков облачных решений Java](/azure/developer/java/).
