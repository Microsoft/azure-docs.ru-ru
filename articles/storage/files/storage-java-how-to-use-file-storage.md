---
title: Разработка для файлов Azure с использованием языка Java | Документы Майкрософт
description: Узнайте, как разрабатывать приложения и службы Java, использующие файлы Azure для хранения файлов данных.
author: roygara
ms.service: storage
ms.topic: how-to
ms.date: 11/18/2020
ms.custom: devx-track-java
ms.author: rogarana
ms.subservice: files
ms.openlocfilehash: 25baa278961b93b04e60f2e997b98753cb6cf3ab
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "95024115"
---
# <a name="develop-for-azure-files-with-java"></a>Разработка для файлов Azure с использованием языка Java

[!INCLUDE [storage-selector-file-include](../../../includes/storage-selector-file-include.md)]

Ознакомьтесь с основами разработки приложений Java, которые используют службы файлов Azure для хранения данных. Создание консольного приложения и изучение основных действий с помощью API службы файлов Azure.

- Создание и удаление общих папок Azure.
- Создание и удаление каталогов.
- Перечисление файлов и каталогов в общей папке Azure
- Передача, загрузка и удаление файлов.

[!INCLUDE [storage-check-out-samples-java](../../../includes/storage-check-out-samples-java.md)]

## <a name="create-a-java-application"></a>Создание приложения Java

Для создания примеров вам потребуется пакет Java Development Kit (JDK) и [пакет SDK для службы хранилища Azure для Java](https://github.com/azure/azure-sdk-for-java). Вам также необходимо создать учетную запись хранилища Azure.

## <a name="set-up-your-application-to-use-azure-files"></a>Настройка приложения для работы со службой файлов Azure

Чтобы использовать API службы файлов Azure, добавьте следующий код в начало файла Java, из которого вы собираетесь получить доступ к службе "файлы Azure".

# <a name="java-v12"></a>[Версия для Java 12](#tab/java)

:::code language="java" source="~/azure-storage-snippets/files/howto/java/java-v12/files-howto-v12/src/main/java/com/files/howto/App.java" id="Snippet_ImportStatements":::

# <a name="java-v11"></a>[Версии 11 Java](#tab/java11)

```java
// Include the following imports to use Azure Files APIs v11
import com.microsoft.azure.storage.*;
import com.microsoft.azure.storage.file.*;
```

---

## <a name="set-up-an-azure-storage-connection-string"></a>Настройка строки подключения к хранилищу Azure

Для использования файлов Azure необходимо подключиться к учетной записи хранения Azure. Настройте строку подключения и используйте ее для подключения к учетной записи хранения. Определите статическую переменную для хранения строки подключения.

# <a name="java-v12"></a>[Версия для Java 12](#tab/java)

Замените *\<storage_account_name\>* и *\<storage_account_key\>* фактическими значениями для вашей учетной записи хранения.

:::code language="java" source="~/azure-storage-snippets/files/howto/java/java-v12/files-howto-v12/src/main/java/com/files/howto/App.java" id="Snippet_ConnectionString":::

# <a name="java-v11"></a>[Версии 11 Java](#tab/java11)

Замените *your_storage_account_name* и *your_storage_account_key* фактическими значениями для вашей учетной записи хранения.

```java
// Configure the connection-string with your values
public static final String storageConnectionString =
    "DefaultEndpointsProtocol=http;" +
    "AccountName=your_storage_account_name;" +
    "AccountKey=your_storage_account_key";
```

---

## <a name="access-azure-files-storage"></a>Доступ к хранилищу файлов Azure

# <a name="java-v12"></a>[Версия для Java 12](#tab/java)

Чтобы получить доступ к службе файлов Azure, создайте объект [шареклиент](/java/api/com.azure.storage.file.share.shareclient) . Используйте класс [шареклиентбуилдер](/java/api/com.azure.storage.file.share.shareclientbuilder) для создания нового объекта **шареклиент** .

:::code language="java" source="~/azure-storage-snippets/files/howto/java/java-v12/files-howto-v12/src/main/java/com/files/howto/App.java" id="Snippet_createClient":::

# <a name="java-v11"></a>[Версии 11 Java](#tab/java11)

Чтобы получить доступ к учетной записи хранения, используйте объект **CloudStorageAccount** , передав строку подключения методу **Parse** .

```java
// Use the CloudStorageAccount object to connect to your storage account
try {
    CloudStorageAccount storageAccount = CloudStorageAccount.parse(storageConnectionString);
} catch (InvalidKeyException invalidKey) {
    // Handle the exception
}
```

**CloudStorageAccount.parse** вызывает прерывание InvalidKeyException, поэтому необходимо поместить его в блок try-catch.

---

## <a name="create-a-file-share"></a>Создание общей папки

Все файлы и каталоги в службе файлов Azure хранятся в контейнере, который называется общей папкой.

# <a name="java-v12"></a>[Версия для Java 12](#tab/java)

Метод [шареклиент. Create](/java/api/com.azure.storage.file.share.shareclient.create) создает исключение, если общая папка уже существует. Установите вызов для **создания** в `try/catch` блоке и обработайте исключение.

:::code language="java" source="~/azure-storage-snippets/files/howto/java/java-v12/files-howto-v12/src/main/java/com/files/howto/App.java" id="Snippet_createFileShare":::

# <a name="java-v11"></a>[Версии 11 Java](#tab/java11)

Чтобы получить доступ к общей папке и ее содержимому, создайте клиент службы файлов Azure.

```java
// Create the Azure Files client.
CloudFileClient fileClient = storageAccount.createCloudFileClient();
```

С его помощью вы получите ссылку на общую папку.

```java
// Get a reference to the file share
CloudFileShare share = fileClient.getShareReference("sampleshare");
```

Чтобы создать общую папку, используйте метод **createIfNotExists** объекта **CloudFileShare** .

```java
if (share.createIfNotExists()) {
    System.out.println("New share created");
}
```

На этом этапе **общий доступ** содержит ссылку на общую папку с именем **Sample Share**.

---

## <a name="delete-a-file-share"></a>Удаление общей папки

В следующем примере кода удаляется общая папка.

# <a name="java-v12"></a>[Версия для Java 12](#tab/java)

Удалите общий ресурс, вызвав метод [шареклиент. Delete](/java/api/com.azure.storage.file.share.shareclient.delete) .

:::code language="java" source="~/azure-storage-snippets/files/howto/java/java-v12/files-howto-v12/src/main/java/com/files/howto/App.java" id="Snippet_deleteFileShare":::

# <a name="java-v11"></a>[Версии 11 Java](#tab/java11)

Удалите общий ресурс, вызвав метод **deleteIfExists** для объекта **CloudFileShare** .

```java
try
{
    // Retrieve storage account from connection-string.
    CloudStorageAccount storageAccount = CloudStorageAccount.parse(storageConnectionString);

    // Create the file client.
   CloudFileClient fileClient = storageAccount.createCloudFileClient();

   // Get a reference to the file share
   CloudFileShare share = fileClient.getShareReference("sampleshare");

   if (share.deleteIfExists()) {
       System.out.println("sampleshare deleted");
   }
} catch (Exception e) {
    e.printStackTrace();
}
```

---

## <a name="create-a-directory"></a>Создание каталога

Организуйте хранение, помещая файлы внутри подкаталогов, вместо того чтобы хранить их в корневом каталоге.

# <a name="java-v12"></a>[Версия для Java 12](#tab/java)

Следующий код создает каталог путем вызова [шаредиректориклиент. Create](/java/api/com.azure.storage.file.share.sharedirectoryclient.create). В примере метода возвращается `Boolean` значение, указывающее, успешно ли был создан каталог.

:::code language="java" source="~/azure-storage-snippets/files/howto/java/java-v12/files-howto-v12/src/main/java/com/files/howto/App.java" id="Snippet_createDirectory":::

# <a name="java-v11"></a>[Версии 11 Java](#tab/java11)

Следующий код создает подкаталог с именем **SampleDir** в корневом каталоге.

```java
//Get a reference to the root directory for the share.
CloudFileDirectory rootDir = share.getRootDirectoryReference();

//Get a reference to the sampledir directory
CloudFileDirectory sampleDir = rootDir.getDirectoryReference("sampledir");

if (sampleDir.createIfNotExists()) {
    System.out.println("sampledir created");
} else {
    System.out.println("sampledir already exists");
}
```

---

## <a name="delete-a-directory"></a>Удаление каталога

Удаление каталога — это простая задача. Нельзя удалить каталог, который по-прежнему содержит файлы или подкаталоги.

# <a name="java-v12"></a>[Версия для Java 12](#tab/java)

Метод [шаредиректориклиент. Delete](/java/api/com.azure.storage.file.share.sharedirectoryclient.delete) создает исключение, если каталог не существует или не является пустым. Установите вызов для **удаления** в `try/catch` блоке и обработайте исключение.

:::code language="java" source="~/azure-storage-snippets/files/howto/java/java-v12/files-howto-v12/src/main/java/com/files/howto/App.java" id="Snippet_deleteDirectory":::

# <a name="java-v11"></a>[Версии 11 Java](#tab/java11)

```java
// Get a reference to the root directory for the share.
CloudFileDirectory rootDir = share.getRootDirectoryReference();

// Get a reference to the directory you want to delete
CloudFileDirectory containerDir = rootDir.getDirectoryReference("sampledir");

// Delete the directory
if ( containerDir.deleteIfExists() ) {
    System.out.println("Directory deleted");
}
```

---

## <a name="enumerate-files-and-directories-in-an-azure-file-share"></a>Перечисление файлов и каталогов в общей папке Azure

# <a name="java-v12"></a>[Версия для Java 12](#tab/java)

Получите список файлов и каталогов, вызвав [шаредиректориклиент. листфилесанддиректориес](/java/api/com.azure.storage.file.share.sharedirectoryclient.listfilesanddirectories). Метод возвращает список объектов [шарефилеитем](/java/api/com.azure.storage.file.share.models.sharefileitem) , для которых можно выполнить итерацию. Следующий код перечисляет файлы и каталоги внутри каталога, указанного параметром *dirName* .

:::code language="java" source="~/azure-storage-snippets/files/howto/java/java-v12/files-howto-v12/src/main/java/com/files/howto/App.java" id="Snippet_enumerateFilesAndDirs":::

# <a name="java-v11"></a>[Версии 11 Java](#tab/java11)

Получите список файлов и каталогов, вызвав **листфилесанддиректориес** в ссылке **клаудфиледиректори** . Метод возвращает список объектов **листфилеитем** , для которых можно выполнить итерацию. Следующий код выводит список файлов и каталогов в корневом каталоге.

```java
//Get a reference to the root directory for the share.
CloudFileDirectory rootDir = share.getRootDirectoryReference();

for ( ListFileItem fileItem : rootDir.listFilesAndDirectories() ) {
    System.out.println(fileItem.getUri());
}
```

---

## <a name="upload-a-file"></a>Отправка файла

Узнайте, как отправить файл из локального хранилища.

# <a name="java-v12"></a>[Версия для Java 12](#tab/java)

Следующий код передает локальный файл в хранилище файлов Azure, вызывая метод [шарефилеклиент. uploadFromFile](/java/api/com.azure.storage.file.share.sharefileclient.uploadfromfile) . В следующем примере метод возвращает `Boolean` значение, указывающее, успешно ли загружен указанный файл.

:::code language="java" source="~/azure-storage-snippets/files/howto/java/java-v12/files-howto-v12/src/main/java/com/files/howto/App.java" id="Snippet_uploadFile":::

# <a name="java-v11"></a>[Версии 11 Java](#tab/java11)

Получите ссылку на каталог, куда будет отправлен файл, вызвав метод **жетрутдиректориреференце** объекта Share.

```java
//Get a reference to the root directory for the share.
CloudFileDirectory rootDir = share.getRootDirectoryReference();
```

Теперь, когда у вас имеется ссылка на корневой каталог общего ресурса, вы можете отправить туда файл с помощью следующего кода.

```java
// Define the path to a local file.
final String filePath = "C:\\temp\\Readme.txt";

CloudFile cloudFile = rootDir.getFileReference("Readme.txt");
cloudFile.uploadFromFile(filePath);
```

---

## <a name="download-a-file"></a>Загрузка файла

Одной из наиболее частых операций является скачивание файлов из хранилища файлов Azure.

# <a name="java-v12"></a>[Версия для Java 12](#tab/java)

В следующем примере загружается указанный файл в локальный каталог, указанный в параметре *destDir* . В примере метод делает скачанное имя файла уникальным, добавляя дату и время.

:::code language="java" source="~/azure-storage-snippets/files/howto/java/java-v12/files-howto-v12/src/main/java/com/files/howto/App.java" id="Snippet_downloadFile":::

# <a name="java-v11"></a>[Версии 11 Java](#tab/java11)

В следующем примере загружается SampleFile.txt и отображается его содержимое.

```java
//Get a reference to the root directory for the share.
CloudFileDirectory rootDir = share.getRootDirectoryReference();

//Get a reference to the directory that contains the file
CloudFileDirectory sampleDir = rootDir.getDirectoryReference("sampledir");

//Get a reference to the file you want to download
CloudFile file = sampleDir.getFileReference("SampleFile.txt");

//Write the contents of the file to the console.
System.out.println(file.downloadText());
```

---

## <a name="delete-a-file"></a>Удаление файла

Другая распространенная операция в службе файлов Azure — это удаление файлов.

# <a name="java-v12"></a>[Версия для Java 12](#tab/java)

Следующий код удаляет указанный файл. Во-первых, в примере создается [шаредиректориклиент](/java/api/com.azure.storage.file.share.sharedirectoryclient) на основе параметра *dirName* . Затем код получает [шарефилеклиент](/java/api/com.azure.storage.file.share.sharefileclient) от клиента каталога на основе параметра *filename* . Наконец, метод example вызывает [шарефилеклиент. Delete](/java/api/com.azure.storage.file.share.sharefileclient.delete) для удаления файла.

:::code language="java" source="~/azure-storage-snippets/files/howto/java/java-v12/files-howto-v12/src/main/java/com/files/howto/App.java" id="Snippet_deleteFile":::

# <a name="java-v11"></a>[Версии 11 Java](#tab/java11)

Следующий пример программы удаляет файл с именем SampleFile.txt, хранящийся в каталоге с именем **sampledir**.

```java
// Get a reference to the root directory for the share.
CloudFileDirectory rootDir = share.getRootDirectoryReference();

// Get a reference to the directory where the file to be deleted is in
CloudFileDirectory containerDir = rootDir.getDirectoryReference("sampledir");

String filename = "SampleFile.txt"
CloudFile file;

file = containerDir.getFileReference(filename)
if ( file.deleteIfExists() ) {
    System.out.println(filename + " was deleted");
}
```

---

## <a name="next-steps"></a>Дальнейшие действия

Если вы хотите узнать больше о других API-интерфейсах Azure, пожалуйста перейдите по следующим ссылкам.

- [Azure for Java developers](/azure/developer/java) (Azure для разработчиков Java);
- [Пакет Azure SDK для Java](https://github.com/azure/azure-sdk-for-java)
- [Пакет Azure SDK для Android](https://github.com/azure/azure-sdk-for-android)
- [Справочник по клиентской библиотеке общей папки Azure для пакета SDK для Java](/java/api/overview/azure/storage-file-share-readme)
- [API-интерфейс REST служб хранилища Azure](/rest/api/storageservices/)
- [Блог рабочей группы службы хранилища Azure](https://azure.microsoft.com/blog/topics/storage-backup-and-recovery/)
- [Приступая к работе со служебной программой командной строки AzCopy](../common/storage-use-azcopy-v10.md)
- [Устранение неполадок с файлами Azure в Windows](storage-troubleshoot-windows-file-connection-problems.md)
