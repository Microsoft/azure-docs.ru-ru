---
title: Пакет SDK для Java — операции файловой системы в Data Lake Storage 1-го поколения — Azure
description: Используйте пакет SDK для Java для Azure Data Lake Storage 1-го поколения выполнения операций файловой системы на Data Lake Storage 1-го поколения таких как создание папок, а также загрузка и скачивание файлов данных.
author: twooley
ms.service: data-lake-store
ms.topic: how-to
ms.date: 05/29/2018
ms.custom: devx-track-java
ms.author: twooley
ms.openlocfilehash: a2c55a2d3277bbb6c3cf72f5ea703780d2a5e9bd
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "87318850"
---
# <a name="filesystem-operations-on-azure-data-lake-storage-gen1-using-java-sdk"></a>Операции файловой системы в Azure Data Lake Storage 1-го поколения с использованием Java SDK
> [!div class="op_single_selector"]
> * [Пакет SDK для .NET](data-lake-store-data-operations-net-sdk.md)
> * [пакет SDK для Java](data-lake-store-get-started-java-sdk.md)
> * [REST API](data-lake-store-data-operations-rest-api.md)
> * [Python](data-lake-store-data-operations-python.md)
>
> 

Узнайте, как использовать пакет SDK для Java Azure Data Lake Storage 1-го поколения для выполнения основных операций, таких как создание папок, отправка и скачивание файлов данных и т. д. Дополнительные сведения о Data Lake Storage 1-го поколения см. в разделе [Azure Data Lake Storage 1-го поколения](data-lake-store-overview.md).

Документацию по API пакета Java SDK для Data Lake Storage 1-го поколения можно найти [здесь](https://azure.github.io/azure-data-lake-store-java/javadoc/).

## <a name="prerequisites"></a>Предварительные требования
* Комплект разработчика Java (JDK 7 или более поздней версии с использованием Java версии 1.7 или более поздней).
* Учетная запись Data Lake Storage 1-го поколения. Следуйте инструкциям из статьи [Начало работы с Azure Data Lake Storage Gen1 с помощью портала Azure](data-lake-store-get-started-portal.md).
* [Maven](https://maven.apache.org/install.html). В этом руководстве это средство используется для создания зависимостей проекта. Хотя зависимости можно создать и без использования таких систем, как Maven или Gradle, они существенно упрощают управление ими.
* Интегрированная среда разработки, например [IntelliJ IDEA](https://www.jetbrains.com/idea/download/), [Eclipse](https://www.eclipse.org/downloads/) или аналогичная (необязательно).

## <a name="create-a-java-application"></a>Создание приложения Java
На [сайте GitHub](https://azure.microsoft.com/documentation/samples/data-lake-store-java-upload-download-get-started/) доступен пример кода, который используется для создания файлов в хранилище, объединения и скачивания файлов, а также для удаления файлов из хранилища. В этом разделе статьи рассматриваются основные части этого кода.

1. Создайте проект Maven с использованием [архетипа mvn](https://maven.apache.org/guides/getting-started/maven-in-five-minutes.html) с помощью командной строки или интегрированной среды разработки (IDE). Инструкции по созданию проекта Java с использованием IntelliJ см. [здесь](https://www.jetbrains.com/help/idea/2016.1/creating-and-running-your-first-java-application.html). Инструкции по созданию проекта с использованием Eclipse см. [здесь](https://help.eclipse.org/mars/index.jsp?topic=%2Forg.eclipse.jdt.doc.user%2FgettingStarted%2Fqs-3.htm). 

2. Добавьте приведенные ниже зависимости в файл Maven **pom.xml**. Добавьте следующий фрагмент перед **\</project>** тегом:
   
    ```xml
    <dependencies>
        <dependency>
        <groupId>com.microsoft.azure</groupId>
        <artifactId>azure-data-lake-store-sdk</artifactId>
        <version>2.1.5</version>
        </dependency>
        <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-nop</artifactId>
        <version>1.7.21</version>
        </dependency>
    </dependencies>
    ```
   
    Первая зависимость предназначена для использования пакета SDK для Data Lake Storage 1-го поколения (`azure-data-lake-store-sdk`) из репозитория Maven. Вторая зависимость нужна, чтобы указать, какие платформы ведения журналов (`slf4j-nop`) будут использоваться для этого приложения. Пакет SDK для Data Lake Storage 1-го поколения использует [SLF4J](https://www.slf4j.org/) Logging фасадной, который позволяет выбирать из ряда популярных платформ ведения журналов, таких как log4j, ведение журнала Java, Logback и т. д., а также без ведения журнала. В этом примере мы отключим ведение журнала, так как используем привязку **slf4j-nop**. Сведения о других вариантах ведения журнала в приложении см. [здесь](https://www.slf4j.org/manual.html#projectDep).

3. Добавьте следующие инструкции импорта в приложение.

    ```java
    import com.microsoft.azure.datalake.store.ADLException;
    import com.microsoft.azure.datalake.store.ADLStoreClient;
    import com.microsoft.azure.datalake.store.DirectoryEntry;
    import com.microsoft.azure.datalake.store.IfExists;
    import com.microsoft.azure.datalake.store.oauth2.AccessTokenProvider;
    import com.microsoft.azure.datalake.store.oauth2.ClientCredsTokenProvider;

    import java.io.*;
    import java.util.Arrays;
    import java.util.List;
    ```

## <a name="authentication"></a>Аутентификация

* Дополнительные сведения о проверке подлинности пользователей в приложении см. в статье [Проверка подлинности пользователей в Data Lake Storage 1-го поколения с использованием Java](data-lake-store-end-user-authenticate-java-sdk.md).
* Дополнительные сведения о проверке подлинности между службами в приложении см. в статье [Проверка подлинности между службами в Data Lake Storage 1-го поколения с использованием Java](data-lake-store-service-to-service-authenticate-java.md).

## <a name="create-a-data-lake-storage-gen1-client"></a>Создание клиента Data Lake Storage 1-го поколения
Создавая объект [ADLStoreClient](https://azure.github.io/azure-data-lake-store-java/javadoc/), укажите имя учетной записи Data Lake Storage 1-го поколения и поставщик маркера, который был создан при проверке подлинности в Data Lake Storage 1-го поколения (см. раздел [Проверка подлинности](#authentication)). Для учетной записи Data Lake Storage 1-го поколения необходимо указать полное доменное имя. Например, замените часть **FILL-IN-HERE** примерно таким значением: **mydatalakestoragegen1.azuredatalakestore.net**.

```java
private static String accountFQDN = "FILL-IN-HERE";  // full account FQDN, not just the account name
ADLStoreClient client = ADLStoreClient.createClient(accountFQDN, provider);
```

Фрагменты кода в следующих разделах содержат примеры некоторых распространенных операций с файловой системой. Другие операции можно найти в полной [документации по API пакета Java SDK для Data Lake Storage 1-го поколения](https://azure.github.io/azure-data-lake-store-java/javadoc/) в разделе об объекте **ADLStoreClient**.

## <a name="create-a-directory"></a>Создание каталога

Следующий фрагмент создает структуру каталогов в корне указанной учетной записи Data Lake Storage 1-го поколения.

```java
// create directory
client.createDirectory("/a/b/w");
System.out.println("Directory created.");
```

## <a name="create-a-file"></a>Создание файла

Следующий фрагмент кода создает файл (c.txt) в структуре каталогов и записывает в него некоторые данные.

```java
// create file and write some content
String filename = "/a/b/c.txt";
OutputStream stream = client.createFile(filename, IfExists.OVERWRITE  );
PrintStream out = new PrintStream(stream);
for (int i = 1; i <= 10; i++) {
    out.println("This is line #" + i);
    out.format("This is the same line (%d), but using formatted output. %n", i);
}
out.close();
System.out.println("File created.");
```

Можно также создать файл (d.txt) с помощью массивов байтов.

```java
// create file using byte arrays
stream = client.createFile("/a/b/d.txt", IfExists.OVERWRITE);
byte[] buf = getSampleContent();
stream.write(buf);
stream.close();
System.out.println("File created using byte array.");
```

Определение функции `getSampleContent`, используемой в предыдущем фрагменте кода, доступно как часть примера [на GitHub](https://azure.microsoft.com/documentation/samples/data-lake-store-java-upload-download-get-started/). 

## <a name="append-to-a-file"></a>Добавление данных в файл

Следующий фрагмент кода добавляет содержимое в имеющийся файл.

```java
// append to file
stream = client.getAppendStream(filename);
stream.write(getSampleContent());
stream.close();
System.out.println("File appended.");
```

Определение функции `getSampleContent`, используемой в предыдущем фрагменте кода, доступно как часть примера [на GitHub](https://azure.microsoft.com/documentation/samples/data-lake-store-java-upload-download-get-started/).

## <a name="read-a-file"></a>Чтение файла

Следующий фрагмент кода считывает содержимое из файла в учетной записи Data Lake Storage 1-го поколения.

```java
// Read File
InputStream in = client.getReadStream(filename);
BufferedReader reader = new BufferedReader(new InputStreamReader(in));
String line;
while ( (line = reader.readLine()) != null) {
    System.out.println(line);
}
reader.close();
System.out.println();
System.out.println("File contents read.");
```

## <a name="concatenate-files"></a>Сцепление файлов

Следующий фрагмент кода сцепляет два файла в учетной записи Data Lake Storage 1-го поколения. При успешном сцеплении объединенный файл заменяет два имеющихся файла.

```java
// concatenate the two files into one
List<String> fileList = Arrays.asList("/a/b/c.txt", "/a/b/d.txt");
client.concatenateFiles("/a/b/f.txt", fileList);
System.out.println("Two files concatenated into a new file.");
```

## <a name="rename-a-file"></a>Переименуйте файл

Следующий фрагмент кода переименовывает файл в учетной записи Data Lake Storage 1-го поколения.

```java
//rename the file
client.rename("/a/b/f.txt", "/a/b/g.txt");
System.out.println("New file renamed.");
```

## <a name="get-metadata-for-a-file"></a>Получение метаданных для файла

Следующий фрагмент кода считывает метаданные из файла в учетной записи Data Lake Storage 1-го поколения.

```java
// get file metadata
DirectoryEntry ent = client.getDirectoryEntry(filename);
printDirectoryInfo(ent);
System.out.println("File metadata retrieved.");
```

## <a name="set-permissions-on-a-file"></a>Установка разрешений для файла

Следующий фрагмент кода задает разрешения для файла, созданного в предыдущем разделе.

```java
// set file permission
client.setPermission(filename, "744");
System.out.println("File permission set.");
```

## <a name="list-directory-contents"></a>Вывод содержимого каталогов

Следующий фрагмент кода рекурсивно отображает содержимое каталога.

```java
// list directory contents
List<DirectoryEntry> list = client.enumerateDirectory("/a/b", 2000);
System.out.println("Directory listing for directory /a/b:");
for (DirectoryEntry entry : list) {
    printDirectoryInfo(entry);
}
System.out.println("Directory contents listed.");
```

Определение функции `printDirectoryInfo`, используемой в предыдущем фрагменте кода, доступно как часть примера [на GitHub](https://azure.microsoft.com/documentation/samples/data-lake-store-java-upload-download-get-started/).

## <a name="delete-files-and-folders"></a>Удаление файлов и папок

Следующий фрагмент кода рекурсивно удаляет указанные файлы и папки из учетной записи хранения Data Lake Storage 1-го поколения.

```java
// delete directory along with all the subdirectories and files in it
client.deleteRecursive("/a");
System.out.println("All files and folders deleted recursively");
promptEnterKey();
```

## <a name="build-and-run-the-application"></a>Создание и запуск приложения
1. Для запуска из среды IDE найдите и нажмите кнопку **запуска**. Для запуска из Maven выполните команду [exec:exec](https://www.mojohaus.org/exec-maven-plugin/exec-mojo.html).
2. Чтобы получить автономный JAR-файл, который можно запустить из командной строки, создайте такой файл со всеми зависимостями с помощью [подключаемого модуля сборки Maven](https://maven.apache.org/plugins/maven-assembly-plugin/usage.html). Пример содержится в файле pom.xml в [исходном коде на сайте GitHub](https://github.com/Azure-Samples/data-lake-store-java-upload-download-get-started/blob/master/pom.xml).

## <a name="next-steps"></a>Дальнейшие действия
* [Документация по пакету SDK для Java](https://azure.github.io/azure-data-lake-store-java/javadoc/)
* [Защита данных в Data Lake Storage Gen1](data-lake-store-secure-data.md)


